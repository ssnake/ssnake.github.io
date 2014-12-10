---
layout: post
title: ASUS N550JK on Ubuntu
comments: true
tag: asus, n550jk, ubuntu, subwoofer, backlight, touchpad
---

Here is list of issues I faced with Ubuntu 14.04

### 1. Hotkeys don’t work

Changing of backlight of keys, sonitor brightness level, sound level

### Fix:
Open `/etc/default/grub`
replace string
`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`
to
`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_osi="`
Run `updae-grub` and reboot.



### 2. No touchpad tab in mouse&touchpad settings

```console 
sudo modprobe -r psmouse
cd /usr/src/
sudo dkms remove psmouse/elantech-v6 --all
sudo wget http://www.ouam.fr/~madko/ubuntu/elantech/psmouse-elantech-v7.tar.bz2 
sudo tar jxvf psmouse-elantech-v7.tar.bz2 
sudo dkms add -m psmouse -v elantech-v7
sudo dkms build -m psmouse -v elantech-v7 
sudo dkms install -m psmouse -v elantech-v7
reboot
```


### 3. Subwoofer works incorrectly

Fix:

Added following line into `/etc/modprobe.d/alsa-base.conf`:

```console
	options snd-hda-intel model=asus-mode4
```
Reboot. If you run sound test via sound settings menu you should hear sound from front left, front right and rear right. Rear right should be sounded via subwoofer. Rear left should be without sound. Despite that if you start using music, left and right speakers will work together with subwooder. To check this 
`speaker-test -c6 -twav`

### 4. Battery icon persistently present

### 5. screen calibration
xrandr --output eDP1 --gamma 0.68:0.7:0.7

### 6. Card Reader doesn;t work

### 7 No usb devices in VirtualBox guest machine

https://www.virtualbox.org/ticket/8873