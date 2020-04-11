---
layout: post
title: Set up a wireless Xbox Elite controller with Kodi on Raspbian Buster
---

Recently I started to play with a Raspberry Pi 4. I wanted to have Kodi on it, and I absolutely do **NOT** want cables 
going all across the living room, form the couch to the TV. So I started to look into using my Xbox controller. It is a
wireless Xbox Elite controller, and I am using Raspbian Buster on the Pi.

Here is how to set it all up.


# Pair the controller
## Spot the MAC address
At this point, I would advise to have your controller OFF, or at least not in pairing mode. It will make it easier to spot it later.

Fire up a terminal and run the following :
```bash
sudo hciconfig hci0 down
sudo hciconfig hci0 up
sudo bluetoothctl
```
Then you will get to a console, most likely with the prefix _bluetooth_.  
There, type `scan on`, Enter, **and wait**.

All the discoverable bluetooth devices that are in range are getting discovered and shown with the green prefix _[NEW]_. When you feel like the utility is done detecting new devices, turn on your controller and putting in pairing mode (long-pressing the little button on the front of the controller, next to the left shoulder). It should appear and be easy to recognize.  
You will see a line in the form of :
```
[NEW] Device F6:1A:AF:FE:FA:0C Xbox Elite Wireless Controller
```
Copy the MAC address (here would be `F6:1A:AF:FE:FA:0C`). Then enter the following commands :
```bash
pair F6:1A:AF:FE:FA:0C
# Wait for pairing: successful
trust F6:1A:AF:FE:FA:0C
connect F6:1A:AF:FE:FA:0C
```
On the last ` connect` command, I got an error 2 or 3 times before getting _connection successful_. So if you get errors, just try again a few times. When you get _connection successful_, the light of the controller should stay on.


# Prepare Kodi
## Install Kodi
```bash
sudo apt-get upgrade
sudo apt-get update
sudo apt-get install kodi
```

## Install required add-ons
```bash
sudo apt-get install kodi-peripheral-joystick kodi-pvr-iptvsimple kodi-inputstream-adaptive kodi-inputstream-rtmp
```
You then need to launch Kodi and Enable the add-ons.

## Enhance the framerate
At this point, you might have noticed Kodi UI is a bit laggy, and wonder why is that so, especially if you're running a 4GB Pi4.
To fix that, open a terminal and run `sudo raspi-config`. There, choose _Advanced Options_ >> _Memory split_. Set it to 320 and press _OK_.

## Make Kodi recognize the bluetooth controller
Kodi is using _libinput_ to handle mice and keyboards, but we need to have libinput ignore the controller :
```bash
echo 'SUBSYSTEM=="input", ATTRS{name}=="Xbox Elite Wireless Controller", KERNEL=="event*", MODE="0666", ENV{LIBINPUT_IGNORE_DEVICE}="1"'

echo 'SUBSYSTEM=="input", ATTRS{name}=="Xbox Elite Wireless Controller", KERNEL=="event*", MODE="0666", ENV{ID_INPUT_JOYSTICK}="1"'

sudo udevadm trigger
```
Note that I used the name `Xbox Elite Wireless Controller`, but if you have a regular Xbox One wireless controller, it would probably be named `Xbox Wireless Controller`. You can check this with the dropdown menu when clicking the bluetooth icon in the taskbar.  
It is also the name shown next to the MAC address when you do a scan in `bluetoothctl`.

# Wrapping it up
If you launch Kodi and press a button on your controller (while it is on of course !), you should get a popup in Kodi, asking you if you want to configure it. Say yes and follow the instructions : press up, press right, etc.
You might get some popups (_Press all analog buttons_), just cancel them and go back to configuring the keys you were doing.
