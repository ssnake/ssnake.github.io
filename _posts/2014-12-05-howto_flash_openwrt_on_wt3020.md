---
layout: post
title: Howto flash OpenWrt on Nexx WT3020
comments: true
tag: [openwrt, flashing, serial, tftp, wt3020]
---

Hi! This is a tutorial of how to flash OpenWrt on WT3020. WT3020 is chinese mini router and it's extremly cheap. You can buy it on Aliexress for ~$15 (for example [here](http://www.aliexpress.com/item/New-Smallest-WT3020A-300M-Portable-Mini-Router-802-11-b-g-n-AP-Repeater-Wifi-Wireless/32217004693.html)).

###!!!Important!!!

The process of flasing goes via serial port. I haven't found another way to flash it( for example, using tftp on router booting). So, you have to open WT3020 box and solder a few connections (TX,RX,GND).

What we need:

* WT3020 router
* TTL-USB converter (cp210, cp2102)
* ubuntu 
* ethernet cable
* openwrt firmware ([link](http://onionwrt.link/download/openwrt-ramips-mt7620n-wt3020-4M-squashfs-sysupgrade.bin))

Useful links (recommend to look thru):

* [OpenWRT: Nexx WT3020](http://wiki.openwrt.org/toh/nexx/wt3020)
* [OpenWRT: generic flashing serial](http://wiki.openwrt.org/doc/howto/generic.flashing.serial)


### Disassembling

Use a screwdriver to make a gap in a place where two upper and bottom lids connected. Gradually and smoothly take apart these two peaces. It should be opened without big efforts. 

<img src="/img/nexx.wt3020a.serial.ttl.jpg">

After openning of the router you need to solder 3 joints. 3 joins represent 3 TTL pins: TX, RX, GND. They are should be connected to TTL-USB converter(cp210, cp2102). Connect pins according the table below:
<center>
	<table>
		<thead>
		<tr>
			<td><h3>Router</h3></td>
			<td><h3>Converter</h3></td>
		</tr>
		</thead>
		<tbody>
		<tr>
			<td>TX</td>
			<td>RX</td>
		</tr>
		<tr>
			<td>RX</td>
			<td>TX</td>
		</tr>
		<tr>
			<td>GND</td>
			<td>GND</td>
		</tr>
		</tbody>
	</table>
</center>

You will need following packages:

{% highlight console %}
sudo apt-get install tftp minicom

{% endhighlight %}

### Connect to the router via serial port

Before connecting find out a port number of your ttl-usb converter, usually it is `/dev/ttyUSB0`.

Start minicom and set up port connection.

{% highlight console %}
minicom -s
{% endhighlight %}

Select `Serial port setup`. Then press `A`, input `/dev/ttyUSB0`. Press `E` set `57600 bps 8N1`. After all save it `Save setup as dfl`. 

Leave minicom running, do not pay attention to `offline` at status line below. Despite this input stream will appear on terminal window. 

Ok, now it's turn for the router. Unplug the router. Connect the router to ttl-usb converter, then connect ttl-usb converter to PC, in a nutshell the router should be connecter to PC.
Plug in WT32020, following data should appear on terminal window:

{% highlight console %}
============================================ 
Ralink UBoot Version: 4.1.0.0
-------------------------------------------- 
ASIC 7620_MP (Port5<->None)
DRAM component: 512 Mbits DDR, width 16
DRAM bus: 16 bit
Total memory: 64 MBytes
Flash component: SPI Flash
Date:Jan  7 2013  Time:11:32:07
============================================ 
icache: sets:512, ways:4, linesz:32 ,total:65536
dcache: sets:256, ways:4, linesz:32 ,total:32768 

 ##### The CPU freq = 580 MHZ #### 
 estimate memory size =64 Mbytes

Please choose the operation: 
   1: Load system code to SDRAM via TFTP. 
   2: Load system code then write to Flash via TFTP. 
   3: Boot system code via Flash (default).
   4: Entr boot command line interface.
   7: Load Boot Loader code then write to Flash via Serial. 
   9: Load Boot Loader code then write to Flash via TFTP. 
{% endhighlight %}
If you see data above then you have done everything correctly. 

### Flashing

* 1. Connect PC and WT3020 with ethernet cable(lan hole in the router). 
* 2. Make sure minicom is started. Unplug and then plug in the router during boot process type `2` to select `2: Load system code then write to Flash via TFTP.`
* 3. Answer `y` for a warning question about flashin nand memory
* 4. Input the router IP address:
`10.10.10.1`
* 5. Input tftp server IP (ip of PC):
`10.10.10.3`
* 6. Name file to dowload:
`code.bin`. The router should got into waiting mode. It's waiting fot tftp server. Let's bring up tftp online.
* 7. Open another terminal tab and download openwrt firmware:

{% highlight console %}
wget http://onionwrt.link/download/openwrt-ramips-mt7620n-wt3020-4M-squashfs-sysupgrade.bin

{% endhighlight %}
* 8. Create and bring up tftp server on PC.

{% highlight console %}
mkdir -p /tmp/tftp/
cp openwrt*.bin /tmp/tftp/code.bin
sudo chmod a+rwx -R /tmp/tftp/
sudo ip addr add 10.10.10.3/24 dev eth0
sudo dnsmasq -d --port=0 --enable-tftp --tftp-root=/tmp/tftp/
{% endhighlight %}
* 9. THe flashing process should be started.

After couple of minutes WT3020 will reboot into openwrt. Enjoy!. Thanks for reading.