---
layout: post
title: Running Ubuntu on ASUS N550JK 
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

This might help:

{% highlight console  %}
sudo modprobe -r psmouse
cd /usr/src/
sudo dkms remove psmouse/elantech-v6 --all
sudo wget http://www.ouam.fr/~madko/ubuntu/elantech/psmouse-elantech-v7.tar.bz2 
sudo tar jxvf psmouse-elantech-v7.tar.bz2 
sudo dkms add -m psmouse -v elantech-v7
sudo dkms build -m psmouse -v elantech-v7 
sudo dkms install -m psmouse -v elantech-v7
reboot
{% endhighlight %}
Links connected with subject:

[Link 1](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1383097?comments=all)

[Link 2](http://mariusmonton.com/?p=489)

### 3. Subwoofer works incorrectly

Fix:

Added following line into `/etc/modprobe.d/alsa-base.conf`:

{% highlight console %}
	options snd-hda-intel model=asus-mode4
{% endhighlight %}
Reboot. If you run sound test via sound settings menu you should hear sound from front left, front right and rear right. Rear right should be sounded via subwoofer. Rear left should be without sound. Despite that if you start using music, left and right speakers will work together with subwooder. To check this 
`speaker-test -c6 -twav`

### 4. Battery icon persistently present
No solution so far


### 5. screen calibration

{% highlight console %}
xrandr --output eDP1 --gamma 0.68:0.7:0.7
{% endhighlight %}

### 6. Card Reader works unstably

### 7. No usb devices in VirtualBox guest machine

VB doesn't support usb 3.0 so far. Look at [https://www.virtualbox.org/ticket/8873](https://www.virtualbox.org/ticket/8873)

### 8. Power Management

ASUS N550JK has discrete graphic card GeForce GTX 850M, also on board it has integrated card Intel HD Graphics 4600. By default Ubuntu 14.04 doesn't switch between cards correctly in order to reduce power consumption. 

### Fix

Luckuly there is a package [Bumblebee](https://wiki.ubuntu.com/Bumblebee). After installation you should notice longer battery life.

{% highlight console %}
sudo apt-get install bumblebee
{% endhighlight %}