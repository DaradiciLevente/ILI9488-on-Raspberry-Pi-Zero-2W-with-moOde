# ILI9488 on Raspberry Pi Zero 2W with moOde Audio

How to use a ILI9488 SPI display on a Raspberry Pi Zero 2W with moOde Audio.

ATTENTION! This procedure only works correctly on 32-bit systems! Unfortunately, I have not yet found a solution to use this display on a 64-bit system!
If anyone has the solution, I would be happy if they would also provide it to me.

First, we need to make the hardware connections between the ILI9488 display and the Raspberry Pi Zero 2W.

These are the hardware connections I used:

<img width="357" alt="pi2ILI9488displayPinout" src="https://github.com/user-attachments/assets/39603568-4b69-46e7-b80c-c274a247b4b1" />

![dd2c4761-919e-4039-92f0-f9e88d33730b~1](https://github.com/user-attachments/assets/1f7e6041-a6c6-485e-9fe7-ecee577caaa8)

After making the hardware connections and the first boot, I did this.

- Edit the file /boot/config.txt  

```sudo nano /boot/config.txt```

- And if you find the following line:

```dtparam=spi=on```

- Put a # in front of that line so that the line looks like this:

```#dtparam=spi=on```

- Next is the installation of the FBCP driver and its configuration for the ILI9488 display.

```cd ~```

```sudo apt update```

```sudo apt install libraspberrypi-dev```

![ili9488_1](https://github.com/user-attachments/assets/2b716828-2851-4461-95ee-59c81c6ac153)

```sudo apt install cmake git build-essential```

![Screenshot 2025-05-31 161133](https://github.com/user-attachments/assets/67ce19e5-9401-4a10-bf79-1f7abb946949)

- After installing the needed tools, clone the project:

```git clone https://github.com/juj/fbcp-ili9341.git```

![Screenshot 2025-05-31 161443](https://github.com/user-attachments/assets/5e03731f-6c42-4fc9-a58b-bbdeb36b4312)

- Enter the fbcp-ili9341 source folder.

```cd fbcp-ili9341```

- Create a new directory named "build".

```mkdir build```

- Change your current directory to the "build" directory you just created.
- This is where the compilation will happen.

```cd build```

- Configure the build using CMake, with specific settings tailored to your hardware.

```cmake -DILI9488=ON -DGPIO_TFT_DATA_CONTROL=24 -DGPIO_TFT_RESET_PIN=25 -DSPI_BUS_CLOCK_DIVISOR=8 -DSTATISTICS=0  ..```

- Explanation of options:

Option	Meaning
-DILI9488=ON	Enables support for the ILI9488 display (default is ILI9341).
-DGPIO_TFT_DATA_CONTROL=24	Uses GPIO 24 as the DC (Data/Command) pin for SPI communication.
-DGPIO_TFT_RESET_PIN=25	Uses GPIO 25 as the Reset pin for the display.
-DSPI_BUS_CLOCK_DIVISOR=8	Sets SPI clock divisor (base 400 MHz ÷ 8 = 50 MHz SPI speed).
-DSTATISTICS=0	Disables printing performance stats to the console.
..	Refers to the parent directory containing the source code.

- Build (compile) the software using the make system generated by CMake.

```make -j```

![Screenshot 2025-05-31 162246](https://github.com/user-attachments/assets/72723046-5a78-40a5-b731-5e7866f434bc)

- At this point we have compiled the FBCP driver for the ILI9488 SPI display.

- If you are using moOde Audio Player, you will need to delay the driver startup so that the moOde graphical interface is already active and only then start the driver. In this case, you will not do the automatic startup from the /etc/rc.local file!

- If you want to use the display for moOde Audio, you will first need to activate it from the moOde Audio web interface:

CONFIGURATION -> SYSTEM-> Peripherals
Display -> ON
Wake on play -> ON

![Screenshot 2025-05-31 171235](https://github.com/user-attachments/assets/107669fa-f7a7-4bec-9661-951e6a766ba3)

- Create a script named:

```sudo nano /home/USER/start_fbcp.sh```

- Which should have the following content:

```
#!/bin/bash
echo "$(date): Script start" >> /home/USER/fbcp.log
sleep 40
echo "$(date): Sleep ended, starting fbcp-ili9341" >> /home/USER/fbcp.log

exec /home/USER/fbcp-ili9341/build/fbcp-ili9341 --fbdev /dev/fb0 --display-rotation=270
```

- This will cause the graphics driver to start 40 seconds after boot, enough time for moOde Audio to start the graphical interface.

### You can download this script directly from the console like this:

```wget https://raw.githubusercontent.com/DaradiciLevente/ILI9488-on-Raspberry-Pi-Zero-2W-with-moOde-Audio/refs/heads/main/start_fbcp.sh```

- Make sure the script is executable:

```chmod +x /home/USER/start_fbcp.sh```

- Create the file:

```sudo nano /etc/systemd/system/fbcp-ili9341.service```

- Add this content to the file:

```
[Unit]
Description=fbcp-ili9341 copy framebuffer for ILI9486/ILI9488
After=multi-user.target

[Service]
Type=simple
ExecStart=/home/levente/start_fbcp.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
- Save the file.

### Or you can download the file directly from the console:
```https://raw.githubusercontent.com/DaradiciLevente/ILI9488-on-Raspberry-Pi-Zero-2W-with-moOde-Audio/refs/heads/main/etc/systemd/system/fbcp-ili9341.service```

- And after that from the console:

```
sudo systemctl daemon-reload
sudo systemctl enable fbcp-ili9341.service
sudo systemctl restart fbcp-ili9341.service
```

- Status check:

```sudo systemctl status fbcp-ili9341.service```

- Check the created log:

```cat /home/USER/fbcp.log```

- Final test:

``sudo reboot``

- The PCM5102A DAC was connected to the Raspberry Pi Zero 2W the following way:

<img width="357" alt="PiZero2W_PCM5102A_PINOUT" src="https://github.com/user-attachments/assets/c0479c4d-bf09-4c8d-a671-9429cb9f83e7" />

![1f8ddabc-b43d-4cc9-b6ff-b85c44f8adf4~1](https://github.com/user-attachments/assets/dcf4e1f2-f6f1-4cd9-8d59-ba102cb1a4e9)

- To enable the PCM5102A DAC in the moOde Audio interface, go to:

CONFIGURE -> SYSTEM -> AUDIO

- And under

```Named I2S device```

- You should choose:

```Generic-2 I2S(i2s-dac)```

![Screenshot 2025-05-31 173557](https://github.com/user-attachments/assets/ba611387-1aed-418a-a912-c0f0db03ef6e)

- Then reboot Audio mode.

- You can see more details in this video:

[![Video preview](https://img.youtube.com/vi/67Y9l3aPVuQ/hqdefault.jpg)](https://youtu.be/67Y9l3aPVuQ)

- If you want the music to start automatically after boot, enable autostart on boot here:

![Screenshot 2025-05-31 174435](https://github.com/user-attachments/assets/2a6dc673-89d2-4d80-9bd0-1e37d7dd0be9)

# For those who don't want to compile the driver 

- In the /bin folder of this repository, I provide the precompiled version of the fbcp driver for the ILI9488 display.

### You can download this binary file from the console like this:

```wget https://raw.githubusercontent.com/DaradiciLevente/ILI9488-on-Raspberry-Pi-Zero-2W-with-moOde-Audio/main/bin/fbcp-ili9341```

### Or direct link from here:

https://raw.githubusercontent.com/DaradiciLevente/ILI9488-on-Raspberry-Pi-Zero-2W-with-moOde-Audio/main/bin/fbcp-ili9341

- After copying it to your system, you will need to make it executable:

```sudo chmod +x fbcp-ili9341```
- After that you can run the executable directly from the console to test it:

```sudo ./fbcp-ili9341```

### If you want to install it in a global location (e.g. /usr/local/bin), you can use:

```sudo mv fbcp-ili9341 /usr/local/bin/```

### After that you can run it anytime with:

```sudo fbcp-ili9341```

# For those who want to change the initial resolution
- I have made my /boot/config.txt file available:

https://github.com/DaradiciLevente/ILI9488-on-Raspberry-Pi-Zero-2W-with-moOde-Audio/blob/main/boot/config.txt

### You can download this file directly from the ssh console with the following command:

```wget https://raw.githubusercontent.com/DaradiciLevente/ILI9488-on-Raspberry-Pi-Zero-2W-with-moOde-Audio/refs/heads/main/boot/config.txt```

Edit the file and change the initial resolution in lines 17,18 and 19.

![Screenshot 2025-06-02 200618](https://github.com/user-attachments/assets/b598c2ff-ae91-4483-9b13-0e13df9f38c4)


# For those who don't want to get too complicated

- I made a script that will auto-compile the file for you:

https://raw.githubusercontent.com/DaradiciLevente/ILI9488-on-Raspberry-Pi-Zero-2W-with-moOde-Audio/refs/heads/main/auto_install_fbcp.sh

### You can download this script directly from the ssh console with the following command:

```wget https://raw.githubusercontent.com/DaradiciLevente/ILI9488-on-Raspberry-Pi-Zero-2W-with-moOde-Audio/refs/heads/main/auto_install_fbcp.sh```

### Then, you can make the file executable with:

```chmod +x auto_install_fbcp.sh```

### Then run the script:

```./auto_install_fbcp.sh```

# Thanks:
- I found the material that helped me solve the problem here: 

https://bytesnbits.co.uk/retropie-raspberry-pi-0-spi-lcd/

### Thanks to the author for the documentation he made available to us!
