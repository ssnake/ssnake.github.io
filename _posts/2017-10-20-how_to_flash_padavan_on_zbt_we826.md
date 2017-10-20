---
layout: post
title: Howto flash Padavan's firmware on ZBT WE826 router
comments: true
tag: [padavan, asus, flashing, WE826, MT7620, router, wifi]
---

![pic]({{ "/img/we826.jpg" | prepend: site.baseurl }})

Recently I've bought (from [here](https://www.aliexpress.com/item/802-11b-g-n-300Mbps-MT7620A-OpenWrt-WiFi-Wireless-Router/32454239568.html?spm=a2g0s.9042311.0.0.Zifj6m)) chinese wifi router ZBT WE826 working on MediaTek(Ralink) MT7620A chipset. 

It comes with OpenWrt BB firmware installed. Its [page](https://wiki.openwrt.org/toh/zbt/we-826?s[]=we826) on openwrt wiki.


I had troubles with my ISP and pppoe. So I decided to flash padavan's firmware. I didn't found any pre-builded images for my router so I made it by myself.

After flashing the router with padavan's firmware I faced with only two minor issues. The sequence of WAN and LANs ports has changed. LAN1 is WAN and WAN is LAN4 now.
The LEDs are  mixed as well. If you know how to fix this please let me know.

If you can stand this then you can download firmware at a link below:

[Padavan's ZBT WE-826 firmware](https://drive.google.com/open?id=0BwPkElwxNv9USkkzQUFZUkJtV1U)


