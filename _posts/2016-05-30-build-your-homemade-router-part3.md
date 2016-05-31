---
published: true
layout: post
title: Build your homemade router (part 3)
---

This post is the third part of the series "Build your homemade router": the [previous part](/2016/05/23/build-your-homemade-router-part2/) covered the system configuration of a minimal 802.11n (2.4Ghz) access point, that part will finally enable 802.11ac.

According to the documentation of the [Airetos AEX-QCA9880-NX](http://www.airetos.com/products/aex-qca9880-nx/), the chipset is fully 802.11ac capable and we should now be able to move from the (crowded) 2.4Ghz to the 5Ghz channels.

Let's ask the system about it:

```
$ iw list
        ...
                Frequencies:
                        * 2412 MHz [1] (20.0 dBm)
                        * 2417 MHz [2] (20.0 dBm)
                        * 2422 MHz [3] (20.0 dBm)
                        * 2427 MHz [4] (20.0 dBm)
                        * 2432 MHz [5] (20.0 dBm)
                        * 2437 MHz [6] (20.0 dBm)
                        * 2442 MHz [7] (20.0 dBm)
                        * 2447 MHz [8] (20.0 dBm)
                        * 2452 MHz [9] (20.0 dBm)
                        * 2457 MHz [10] (20.0 dBm)
                        * 2462 MHz [11] (20.0 dBm)
                        * 2467 MHz [12] (disabled)
                        * 2472 MHz [13] (disabled)
                        * 2484 MHz [14] (disabled)
        ...
                Frequencies:
                        * 5180 MHz [36] (17.0 dBm) (no IR)
                        * 5200 MHz [40] (17.0 dBm) (no IR)
                        * 5220 MHz [44] (17.0 dBm) (no IR)
                        * 5240 MHz [48] (17.0 dBm) (no IR)
                        * 5260 MHz [52] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5280 MHz [56] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5300 MHz [60] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5320 MHz [64] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5500 MHz [100] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5520 MHz [104] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5540 MHz [108] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5560 MHz [112] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5580 MHz [116] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5600 MHz [120] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5620 MHz [124] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5640 MHz [128] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5660 MHz [132] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5680 MHz [136] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5700 MHz [140] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5720 MHz [144] (23.0 dBm) (no IR, radar detection)
                          DFS state: usable (for 74 sec)
                          DFS CAC time: 60000 ms
                        * 5745 MHz [149] (30.0 dBm) (no IR)
                        * 5765 MHz [153] (30.0 dBm) (no IR)
                        * 5785 MHz [157] (30.0 dBm) (no IR)
                        * 5805 MHz [161] (30.0 dBm) (no IR)
                        * 5825 MHz [165] (30.0 dBm) (no IR)
       ...
```

From the (curated) output above, we can see that the chipset supports channels 1 to 14 (2.4Ghz) and channels 36 to 165 (5Ghz) - but what's that `no IR` flag? 

The `no IR` flag stands for _no-initiating-radiation_ and that means **you cannot use that mode of operation if that require you to initiate radiation first**, and that includes **beacons** (of course). In other words: **you cannot run an access-point on those channels!**

![](http://gph.to/20LFaSu){: .center-image }


## Regulatory compliance

The above situation is because of the [Linux regulatory compliance](https://wireless.wiki.kernel.org/en/developers/regulatory/statement), which regulates usage of the radio spectrum [depending on a territory basis](https://en.wikipedia.org/wiki/List_of_WLAN_channels).

But, wait! I'm living in the US and according to the FCC, I should be able to emit on channels 36-48, so what's wrong? Let's take a look at the regulatory domain currently in use:

```shell
$ iw reg get
country 00: DFS-UNSET
        (2402 - 2472 @ 40), (N/A, 20), (N/A)
        (2457 - 2482 @ 40), (N/A, 20), (N/A), NO-IR
        (2474 - 2494 @ 20), (N/A, 20), (N/A), NO-OFDM, NO-IR
        (5170 - 5250 @ 80), (N/A, 20), (N/A), NO-IR
        (5250 - 5330 @ 80), (N/A, 20), (0 ms), DFS, NO-IR
        (5490 - 5730 @ 160), (N/A, 20), (0 ms), DFS, NO-IR
        (5735 - 5835 @ 80), (N/A, 20), (N/A), NO-IR
        (57240 - 63720 @ 2160), (N/A, 0), (N/A)
```

The output above tell us that the current regulatory domain in use is [_worldwide_](http://linuxwireless.org/en/users/Drivers/ath/#EEPROM_world_regulatory_domain) (or unset), that mean it is currently using minimum values allowed in every country.

Unfortunately, trying to manually set the regulatory domain through `sudo iw reg set US` won't work because the card has been shipped with the [_world regulatory domain_](https://wireless.wiki.kernel.org/en/users/drivers/ath#eeprom_world_regulatory_domain) burned into EEPROM, and all Atheros custom world regulatory domains have all 5 GHz channels marked with a passive scan flags:

```shell
$ dmesg | grep EEPROM
[   12.123068] ath: EEPROM regdomain: 0x6c
```

![](http://i.giphy.com/bcEKH8GjRJovm.gif){: .center-image }

## Patch it!

Fortunately, the regulatory compliance is dealed at the driver level and can be easily patched. The original patch can be found in the [Open-WRT source tree](https://dev.openwrt.org/browser/trunk/package/kernel/mac80211/patches/402-ath_regd_optional.patch).

First of all, be sure to enable the source repository from _/etc/apt/sources.list_:

```shell
$ cat /etc/apt/sources.list
...
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial main restricted 
...
```

Then, prepare the build environment by installing the required dependency:

```
$ VERSION=$(uname -r)
$ sudo apt-get build-dep linux-image-$VERSION
```

Grab the source of your current kernel:

```shell
$ apt-get source linux-image-$VERSION
```

Since the original Open-WRT patch can't be applied "as is" against our Ubuntu tree, I had to _slightly_ [modify it](https://gist.github.com/renaudcerrato/ba9e200af202bb4f651fd2ba09adea6b): 

```shell
$ cd linux-${VERSION%%-*}
$ wget -O - https://gist.github.com/renaudcerrato/ba9e200af202bb4f651fd2ba09adea6b/raw/ab36b11bb0c6357cc0513b2c6500a1841c8dd252/402-ath_regd_optional.patch | patch -p1 -b
```

Eveything should now be ready for the build:

```shell
$ mkdir ~/build && cp /boot/config-$VERSION  ~/build/.config && cp /usr/src/linux-headers-${VERSION}/Module.symvers ~/build/
$ make O=~/build oldconfig
$ make O=~/build prepare
$ make O=~/build scripts
$ make O=~/build SUBDIRS=drivers/net/wireless/ath modules
```

If everything goes well, we can now copy our patched driver over the previous one: 

```shell
$ sudo install -b ~/build/drivers/net/wireless/ath/ath.ko /lib/modules/${VERSION}/kernel/drivers/net/wireless/ath/ath.ko
$ sudo depmod -a
```

Reboot, and voilà!


```shell
$ sudo iw reg set US
$ iw list
...
                Frequencies:
                        * 5180 MHz [36] (17.0 dBm)
                        * 5200 MHz [40] (17.0 dBm)
                        * 5220 MHz [44] (17.0 dBm)
                        * 5240 MHz [48] (17.0 dBm)
                        * 5260 MHz [52] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5280 MHz [56] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5300 MHz [60] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5320 MHz [64] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5500 MHz [100] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5520 MHz [104] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5540 MHz [108] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5560 MHz [112] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5580 MHz [116] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5600 MHz [120] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5620 MHz [124] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5640 MHz [128] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5660 MHz [132] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5680 MHz [136] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5700 MHz [140] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5720 MHz [144] (23.0 dBm) (radar detection)
                          DFS state: usable (for 3470 sec)
                          DFS CAC time: 60000 ms
                        * 5745 MHz [149] (30.0 dBm)
                        * 5765 MHz [153] (30.0 dBm)
                        * 5785 MHz [157] (30.0 dBm)
                        * 5805 MHz [161] (30.0 dBm)
                        * 5825 MHz [165] (30.0 dBm)
...
```

## Configuration

Our new hostapd configuration file should be (almost) straightforward now: `hw_mode=a` will enable the 5Ghz bands and `ieee80211ac=1` will enable 802.11ac (VHT). Using `ieee80211d=1` (along with `country_code=US`), we'll advertise the regulatory domain we're working on.

To get the most of our bandwidth, `ht_capab` and `vht_capab` should been set to reflect the capabilities of the hardware:

```
$ iw list
...
        Band 1:
                Capabilities: 0x19e3
                        RX LDPC
                        HT20/HT40
                        Static SM Power Save
                        RX HT20 SGI
                        RX HT40 SGI
                        TX STBC
                        RX STBC 1-stream
                        Max AMSDU length: 7935 bytes
                        DSSS/CCK HT40
...
        Band 2:
                Capabilities: 0x19e3
                        RX LDPC
                        HT20/HT40
                        Static SM Power Save
                        RX HT20 SGI
                        RX HT40 SGI
                        TX STBC
                        RX STBC 1-stream
                        Max AMSDU length: 7935 bytes
                        DSSS/CCK HT40
...
```

According to the output above, **here's our final _hostapd.conf_**:

```shell
$ cat /etc/hostapd/hostapd.conf
#### Interface configuration ####

interface=wlp5s0
bridge=br0
driver=nl80211

##### IEEE 802.11 related configuration #####

ssid=iCanHearYouHavingSex
hw_mode=a
channel=0
auth_algs=1
wmm_enabled=1
country_code=US
ieee80211d=1
ieee80211h=0

##### IEEE 802.11n related configuration #####

ieee80211n=1
ht_capab=[HT40+][SHORT-GI-40][TX-STBC][RX-STBC1][DSSS_CK-40][LDPC][MAX-AMSDU-7935]

##### IEEE 802.11ac related configuration #####

ieee80211ac=1
vht_capab=[MAX-MPDU-11454][RXLDPC][SHORT-GI-80][TX-STBC-2BY1][RX-STBC-1][MAX-A-MPDU-LEN-EXP7][TX-ANTENNA-PATTERN][RX-ANTENNA-PATTERN]
vht_oper_chwidth=1

##### WPA/IEEE 802.11i configuration #####

wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_passphrase=YouCantGuess
```


> You’ll find the documentation of every available options into the [/usr/share/doc/hostapd/examples/](https://gist.github.com/renaudcerrato/db053d96991aba152cc17d71e7e0f63c) directory.
