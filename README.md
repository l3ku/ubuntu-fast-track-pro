# How to fix Ubuntu 18.04 not recognizing M-Audio Fast Track Pro USB audio interface
After buying a new machine and installing Ubuntu 18.04 on it, I realized that my old Fast Track Pro USB audio interface was not recognized at all by the system as it did not appear in the system preferences. I was not willing to buy a new sound card yet, so as somewhat a Linux noob, I started troubleshooting and eventually got the sound card to work.

Hopefully this saves you from a couple hours of frustration. :)

## 1. Check if your device is recognized by ALSA
```
$ aplay -l
...
card 5: Pro [FastTrack Pro], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 5: Pro [FastTrack Pro], device 1: USB Audio [USB Audio #1]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```
If the command output includes a similar output to the above, it means that ALSA recognizes the card and you can **skip to step 4**.

## 2. Verify that the device is listed as an USB device
```
$ lsusb
...
Bus 001 Device 006: ID 0763:2012 Midiman M-Audio Fast Track Pro
...
```
The output of the command `lsusb` should include the output above. If not, check that the device is connected. If the problem persists, there is some other issue with your system that I can not unfortunately help you any further with.

## 3. Edit ALSA configuration
Open the configuration file `/etc/modprobe.d/alsa-base.conf`:
```
$ sudo vim /etc/modprobe.d/alsa-base.conf
```

Comment out the following line:
```
...
#options snd-usb-audio index=-2
...
```

Add the following line to the end of the file:
```
...
options snd_usb_audio vid=0x763 pid=0x2012 device_setup=0x9 index=5 enable=1
```

For the changes to take effect, reload ALSA:
```
$ sudo alsa force-reload
```
Now you can verify according to step 1 that the device is listed with `aplay -l`.
The sound card should also be listed in output of the following command:
```
$ cat /proc/asound/cards
...
 5 [Pro            ]: USB-Audio - FastTrack Pro
                      M-Audio FastTrack Pro at usb-0000:00:14.0-12, full speed
```

## 4. Edit Pulseaudio configuration
Open the Pulseaudio configuration file for editing:
```
$ sudo vim /etc/pulse/default.pa
```

Add the following line to the end of the file:
```
...
load-module module-alsa-sink device=hw:5
```
**IMPORTANT:** Change the device number from 5 to the actual card number outputted when you run the command `cat /proc/asound/cards`!

Reload Pulseaudio for the changes to take effect:
```
$ pulseaudio -k && pulseaudio
```

## 5. Select Fast Track Pro as sound card
This can be done various ways, I myself used the system settings GUI. If your Fast Track Pro is still not listed in the available devices, try running:
```
pacmd unload-module module-udev-detect && pacmd load-module module-udev-detect
```
If there are some errors or something could be done better, suggestions are welcome in the comment section!
