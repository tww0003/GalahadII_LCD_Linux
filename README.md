# GalahadII LCD Linux
A simple PyUSB script to stream a gif to the Lian-Li Galahad II LCD on Linux - because Linux deserves pretty things too.
This script was created and tested on CachyOS - with some help from a Windows VM, USBPcap, and DNSpyEx for reversing L-Connect.  

## Fork Notes

There appears to be slight differences between Lian-Li hardware revisions that cause compatibility issues with the original script.  
As a result, I forked the repository and updated it to work with my hardware (`LianLi-GA_II-LCD_v1.4`).  
  
The real change was updating the `REPORT_ID_VIDEO` variable to `0x02` from `0x03`. 
If this script isn't working, I recommend enumerating the values for `REPORT_ID_VIDEO` until you find a number that works.
  
Other changes include updating the `README` to fit my preferences, adding a `requirements.txt` file to make resolving dependencies easier, and adding a simple `.gitignore` file.
If I feel motivated I may add more features to this project.

## Setup

### USB Permissions  

First identify/confirm that the GAII device is detected.
Make note of the ID value - 0416:7395 , these are set as the script default but may require changing!

```
-$ lsusb | grep -i 'LianLi-GA'
Bus 001 Device 023: ID 0416:7395 Winbond Electronics Corp. LianLi-GA_II-LCD_v1.6
```

Next - Create a `udev` rule for the USB device and set the permissions, changing your vendor and product id based on the earlier `lsusb` output

```
-$ echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="0416", ATTR{idProduct}=="7395", MODE="0666"' > 99-galahad.rules
-$ sudo mv ./99-galahad.rules /etc/udev/rules.d/
```

Then update the rules with `udevadm`

```
-$ sudo udevadm control --reload-rules
-$ sudo udevadm trigger
```

### Environment  

Create a virtual environment for the script to run in (optional):  

```
-$ python3 -m venv venv
-$ source venv/bin/activate
```
  
Install the dependencies:
```
-$ pip install -r requirements.txt
```

## Usage

Then run the script with your chosen gif, use -h to list args for rotating the GIF, changing VID/PID and frame interval speed.

```
-$ python3 galahadII_LCD.py -i frieren.gif
[+] Device Initialized
[!] Opening frieren.gif...
[!] Saving 40 frames to rotated.gif...
[+] Done!
[!] Converting rotated.gif -> video.h264...
[!] Detected FPS: 10
[+] Conversion Complete.
[+] Streaming 'rotated.gif'...
```

## Systemd Service

Create a new service file called `lconlcd.service` within `/etc/systemd/system/` with the following contents:

```
[Unit]
Description=LianLiGalahadIILCD

[Service]
WorkingDirectory=/opt/GalahadII_LCD_Linux/
ExecStart=/usr/bin/python3 /opt/GalahadII_LCD_Linux/galahadII_LCD.py -i frieren.gif

[Install]
WantedBy=multi-user.target
```

Then perform a daemon-reload and enable the service

```
-$ sudo systemctl daemon-reload
-$ sudo systemctl enable --now lconlcd
```

## Notes

### Original Notes
If interrupted you only have a few seconds to restart the same or a new stream, else when running again the LCD will restart and kill the script.

I recommended that one gif be chosen at a time and this enabled as a service to run on a delay after startup.

This is far from perfect :) - the script basically just throws the H264 data at the LCD without the polling and other comms normally made by L-Connect.
Still - better than staring at the damn Lian-Li logo all day.

### My notes (tww0003)  
  
I haven't encountered the issues with interrupting the script. In my experience, the LCD screen will just freeze on the last frame displayed.  
I also didn't experience any issues swapping between gifs at any point like the original author mentions.
