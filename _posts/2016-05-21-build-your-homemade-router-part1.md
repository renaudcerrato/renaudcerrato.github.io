---
published: true
title: Build your homemade router (part 1)
layout: post
---
I've spent the last decade flashing my home routers to DD-WRT but was never totally satisfied about it because of the cheap (and poorly supported) hardware, unstable builds, and/or unfixed bugs. Not even talking of [the controversy](http://www.wi-fiplanet.com/columns/article.php/3816236/The-DD-WRT-Controversy.htm) about it.  

Since I needed to upgrade my [Buffalo WHR-300HP2](http://www.buffalotech.com/products/wireless/single-band/airstation-highpower-n300-wireless-router-2) to something able to get the most of my 200Mbps cable connection, I recently decided to build my own Ubuntu 16.04LTS based router.

## Bill of Material

- [Gigabyte GA-J1900N-D3V](http://www.amazon.com/Gigabyte-Built-Celeron-Motherboard-GA-J1900N-D3V/dp/B00IW99S4A) (mini-ITX, J1900 Quad-Core 2Ghz Celeron, dual LAN)
- [Airetos AEX-QCA9880-NX](http://www.amazon.com/AIRETOS-AEX-QCA9880-NX-802-11ac-Extended-Temperature/dp/B00OJPJVV6) (dual band 802.11ac, MIMO, ah10k support)
- [4GB RAM](http://www.amazon.com/Crucial-DDR3-1333-PC3-10600-CT2K2G3S1339M-CT2C2G3S1339M/dp/B008LTBIGW) (DDR3-LP, 1333Mhz, 1.35v)
- [mPCIe Extender](http://www.amazon.com/KZ-B22-mini-Express-MiniCard-Extender/dp/B008P1I28I)
- [MX500 mini-ITX case](http://www.amazon.com/MITXPC-MX500-Industrial-Mini-ITX-WallMount/dp/B01B575EMA)
- [3 x 6dBi RPSMA Dual Band Antenna](http://www.amazon.com/Super-Power-Supply%C2%AE-WZR-HP-G450H-TL-WR1043ND/dp/B00E9DN2D6) + [RP-SMA Pigtail Cable](http://www.amazon.com/Super-Power-Supply%C2%AE-Wireless-WN2500RP/dp/B00ITWDN32)
- [PicoPSU-90](http://www.amazon.com/PicoPSU-90-Adapter-Power-Kit-Cyncronix/dp/B00316T5S8)
- Spare 2.5" HDD

## Assembly

The motherboard, RAM and the Pico-PSU installation went smoothly: the case is spacious, and even have (perfectly sized) pre-cut holes for the AC/DC plug:


![assembly](/static/img/assembly-small.jpg)


The trickiest part was the [mini-PCIe Wi-Fi card](http://www.amazon.com/AIRETOS-AEX-QCA9880-NX-802-11ac-Extended-Temperature/dp/B00OJPJVV6): the motherboard doesn't have enough room to support full-size cards, only half-sized. Here come the [mPCIe Extender](http://www.amazon.com/KZ-B22-mini-Express-MiniCard-Extender/dp/B008P1I28I) to the rescue:

![mPCIe extender](/static/img/airetos-small.jpg)

![mPCIe port](/static/img/mpci.jpg)

I choosed a 20cm FFC cable (included) to connect both sides of the adapter and fixed the mini-PCIe side to the chassis using some double sided tape:

![mPCIe port](/static/img/mpci2.jpg)

Since that mini-ITX case comes with 3 (again, perfectly sized) pre-cut holes for your antenna needs, I won't go into details about them. Here's the final result:

![](/static/img/case1.jpg)

![](/static/img/case2.jpg)

## Installation

I choosed [Lubuntu 16.04LTS](http://lubuntu.net): an Ubuntu based distro with a lightweight desktop environment (if ever needed).


The [next part](/2016-05-23-build-your-homemade-router-part2) will configure your freshly installed router.
