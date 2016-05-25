---
published: false
title: Build your homemade router (part 2)
layout: post
---

This post is the second part of the series "Build your homemade router". The [first part](https://renaudcerrato.github.io/2016/05/21/build-your-homemade-router-part1/) covered the material installation.

That second part will cover the configuration of your freshly installed machine and, before going further, we'll need draw what we want to achieve. First of all, let's see what are the name of the interfaces:

```bash
$ ifconfig
enp1s0    Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx           
          ...
enp2s0    Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx  
          ...
wlp5s0    Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx
          ...         
```

The motherboard have 2 built-in LAN, named `enp1s0` and `enp2s0`. The mini-PCIe WiFi card is showing up as `wlp5s0`, and support AP mode as expected:

```shell
$ iw list
...
Supported interface modes:
                 * managed
                 * AP
                 * AP/VLAN
                 * monitor
                 * mesh point
```


## Diagram

From the informations above, we can finally draw our diagram: the first NIC will serve as our WAN port, while the second one will be bridged to our wireless network:


![](/static/img/network-diagram.jpg)



## Configuration

Let's first install the requirements! We'll make use of [dnsmasq](http://manpages.ubuntu.com/manpages/xenial/man8/dnsmasq.8.html) as our DHCP/DNS server

```shell
$ sudo apt-get install dnsmasq bridge-utils
```

We'll need to edit our [network interface configuration](http://manpages.ubuntu.com/manpages/xenial/man5/interfaces.5.html) to match our diagram. Here's a _preliminary_ configuration including a minimal `dnsmasq` setup:

```shell
$ cat /etc/network/interfaces
# Loopback
auto lo
iface lo inet loopback

# WAN interface
auto enp1s0
iface enp1s0 inet dhcp

# Bridge (LAN)
auto br0 
iface br0 inet static
    address 192.168.1.1
    network 192.168.1.0
    netmask 255.255.255.0
    broadcast 192.168.1.255 
    bridge_ports enp2s0 wlp5s0
    post-up /usr/sbin/dnsmasq \
    			--pid-file=/var/run/dnsmasq-br0.pid \
                --conf-file=/dev/null \
    			--interface=br0 --except-interface=lo \
                --dhcp-range=192.168.1.10,192.168.1.150,24h
    pre-down cat /var/run/dnsmasq-br0.pid | xargs kill
```

You can now restart your networking (`sudo service networking restart`) or simply reboot your router to check if your network configuration is properly setup. 

However, please note that while you should be able to get DHCP leases from `enp2s0` at this point, **you won't be able to connect wirelessly** (yet), **nor able to reach internet**.


### hostapd
TODO


Conflicting NetworkManager:
http://askubuntu.com/questions/472794/hostapd-error-nl80211-could-not-configure-driver-mode

Solution:

```
sudo apt-get remove network-manager
```

Dnsmasq + bridge: unknown network interface
https://bugs.launchpad.net/ubuntu/+source/dnsmasq/+bug/1531184


ath10k patches for 5Ghz:
https://dev.openwrt.org/browser/trunk/package/kernel/mac80211/patches
