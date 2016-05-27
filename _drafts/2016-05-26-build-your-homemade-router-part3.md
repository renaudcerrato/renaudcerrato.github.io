---
published: false
layout: post
title: Build your homemade router (part 3)
---

This post is the third part of the series "Build your homemade router": the [previous part](/2016/05/23/build-your-homemade-router-part2/) covered the system configuration of a very basic 802.11n/2.4Ghz access point.

According to the documentation of the [Airetos AEX-QCA9880-NX](http://www.airetos.com/products/aex-qca9880-nx/), the chipset is fully 802.11ac capable and we should now be able to move from the (crowded) 2.4Ghz to the nirvana (a.k.a 5Ghz channels).

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

![](http://gph.to/20LFaSu)


## Regulatory compliance

The above situation is due to the [Linux regulatory compliance](https://wireless.wiki.kernel.org/en/developers/regulatory/statement), which regulate usage of the radio spectrum [depending on a territory basis](https://en.wikipedia.org/wiki/List_of_WLAN_channels).

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

The output above tell us that the current regulatory domain in use is [_worldwide_](http://linuxwireless.org/en/users/Drivers/ath/#EEPROM_world_regulatory_domain), that mean it is currently using minimum values allowed in every country - thus disallowing emission-first on the 5GHz bands.


VERSION=$(uname -r)
sudo apt-get build-dep linux-image-$VERSION

mkdir ~/build
cp /boot/config-$VERSION  ~/build/.config && cp /usr/src/linux-headers-${VERSION}/Module.symvers ~/build/

## Compilation

apt-get source linux-image-$VERSION

cd linux-${VERSION%%-*}

curl -L https://gist.github.com/renaudcerrato/ba9e200af202bb4f651fd2ba09adea6b/raw/ab36b11bb0c6357cc0513b2c6500a1841c8dd252/402-ath_regd_optional.patch | patch -p1 -b

make O=~/build oldconfig && \
make O=~/build prepare && \
make O=~/build scripts && \
make O=~/build SUBDIRS=drivers/net/wireless/ath modules

## Installation

cd ~/build && sudo find . -wholename *drivers/net/wireless/ath*.ko -exec install -b {} /lib/modules/${VERSION}/kernel/{} \; && sudo depmod -a

## Avant 
 ```shell
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

## Aprés

```shell
$ iw list
Wiphy phy0
        max # scan SSIDs: 16
        max scan IEs length: 195 bytes
        Retry short limit: 7
        Retry long limit: 4
        Coverage class: 0 (up to 0m)
        Device supports RSN-IBSS.
        Device supports AP-side u-APSD.
        Supported Ciphers:
                * WEP40 (00-0f-ac:1)
                * WEP104 (00-0f-ac:5)
                * TKIP (00-0f-ac:2)
                * CCMP (00-0f-ac:4)
                * CMAC (00-0f-ac:6)
        Available Antennas: TX 0x7 RX 0x7
        Configured Antennas: TX 0x7 RX 0x7
        Supported interface modes:
                 * managed
                 * AP
                 * AP/VLAN
                 * monitor
                 * mesh point
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
                Maximum RX AMPDU length 65535 bytes (exponent: 0x003)
                Minimum RX AMPDU time spacing: 8 usec (0x06)
                HT TX/RX MCS rate indexes supported: 0-23
                VHT Capabilities (0x338001b2):
                        Max MPDU length: 11454
                        Supported Channel Width: neither 160 nor 80+80
                        RX LDPC
                        short GI (80 MHz)
                        TX STBC
                        RX antenna pattern consistency
                        TX antenna pattern consistency
                VHT RX MCS set:
                        1 streams: MCS 0-9
                        2 streams: MCS 0-9
                        3 streams: MCS 0-9
                        4 streams: not supported
                        5 streams: not supported
                        6 streams: not supported
                        7 streams: not supported
                        8 streams: not supported
                VHT RX highest supported: 0 Mbps
                VHT TX MCS set:
                        1 streams: MCS 0-9
                        2 streams: MCS 0-9
                        3 streams: MCS 0-9
                        4 streams: not supported
                        5 streams: not supported
                        6 streams: not supported
                        7 streams: not supported
                        8 streams: not supported
                VHT TX highest supported: 0 Mbps
                Bitrates (non-HT):
                        * 1.0 Mbps
                        * 2.0 Mbps (short preamble supported)
                        * 5.5 Mbps (short preamble supported)
                        * 11.0 Mbps (short preamble supported)
                        * 6.0 Mbps
                        * 9.0 Mbps
                        * 12.0 Mbps
                        * 18.0 Mbps
                        * 24.0 Mbps
                        * 36.0 Mbps
                        * 48.0 Mbps
                        * 54.0 Mbps
                Frequencies:
                        * 2412 MHz [1] (30.0 dBm)
                        * 2417 MHz [2] (30.0 dBm)
                        * 2422 MHz [3] (30.0 dBm)
                        * 2427 MHz [4] (30.0 dBm)
                        * 2432 MHz [5] (30.0 dBm)
                        * 2437 MHz [6] (30.0 dBm)
                        * 2442 MHz [7] (30.0 dBm)
                        * 2447 MHz [8] (30.0 dBm)
                        * 2452 MHz [9] (30.0 dBm)
                        * 2457 MHz [10] (30.0 dBm)
                        * 2462 MHz [11] (30.0 dBm)
                        * 2467 MHz [12] (disabled)
                        * 2472 MHz [13] (disabled)
                        * 2484 MHz [14] (disabled)
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
                Maximum RX AMPDU length 65535 bytes (exponent: 0x003)
                Minimum RX AMPDU time spacing: 8 usec (0x06)
                HT TX/RX MCS rate indexes supported: 0-23
                VHT Capabilities (0x338001b2):
                        Max MPDU length: 11454
                        Supported Channel Width: neither 160 nor 80+80
                        RX LDPC
                        short GI (80 MHz)
                        TX STBC
                        RX antenna pattern consistency
                        TX antenna pattern consistency
                VHT RX MCS set:
                        1 streams: MCS 0-9
                        2 streams: MCS 0-9
                        3 streams: MCS 0-9
                        4 streams: not supported
                        5 streams: not supported
                        6 streams: not supported
                        7 streams: not supported
                        8 streams: not supported
                VHT RX highest supported: 0 Mbps
                VHT TX MCS set:
                        1 streams: MCS 0-9
                        2 streams: MCS 0-9
                        3 streams: MCS 0-9
                        4 streams: not supported
                        5 streams: not supported
                        6 streams: not supported
                        7 streams: not supported
                        8 streams: not supported
                VHT TX highest supported: 0 Mbps
                Bitrates (non-HT):
                        * 6.0 Mbps
                        * 9.0 Mbps
                        * 12.0 Mbps
                        * 18.0 Mbps
                        * 24.0 Mbps
                        * 36.0 Mbps
                        * 48.0 Mbps
                        * 54.0 Mbps
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
        Supported commands:
                 * new_interface
                 * set_interface
                 * new_key
                 * start_ap
                 * new_station
                 * new_mpath
                 * set_mesh_config
                 * set_bss
                 * authenticate
                 * associate
                 * deauthenticate
                 * disassociate
                 * join_ibss
                 * join_mesh
                 * remain_on_channel
                 * set_tx_bitrate_mask
                 * frame
                 * frame_wait_cancel
                 * set_wiphy_netns
                 * set_channel
                 * set_wds_peer
                 * probe_client
                 * set_noack_map
                 * register_beacons
                 * start_p2p_device
                 * set_mcast_rate
                 * channel_switch
                 * Unknown command (104)
                 * connect
                 * disconnect
        Supported TX frame types:
                 * IBSS: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                 * managed: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                 * AP: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                 * AP/VLAN: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                 * mesh point: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                 * P2P-client: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                 * P2P-GO: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                 * P2P-device: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
        Supported RX frame types:
                 * IBSS: 0x40 0xb0 0xc0 0xd0
                 * managed: 0x40 0xd0
                 * AP: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
                 * AP/VLAN: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
                 * mesh point: 0xb0 0xc0 0xd0
                 * P2P-client: 0x40 0xd0
                 * P2P-GO: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
                 * P2P-device: 0x40 0xd0
        software interface modes (can always be added):
                 * AP/VLAN
                 * monitor
        valid interface combinations:
                 * #{ AP, mesh point } <= 8,
                   total <= 8, #channels <= 1, STA/AP BI must match
        HT Capability overrides:
                 * MCS: ff ff ff ff ff ff ff ff ff ff
                 * maximum A-MSDU length
                 * supported channel width
                 * short GI for 40 MHz
                 * max A-MPDU length exponent
                 * min MPDU start spacing
        Device supports TX status socket option.
        Device supports HT-IBSS.
        Device supports SAE with AUTHENTICATE command
        Device supports scan flush.
        Device supports per-vif TX power setting
        Driver supports a userspace MPM
        Driver/device bandwidth changes during BSS lifetime (AP/GO mode)
        Device supports static SMPS
```

## hostapd.conf

```shell
#### Interface configuration ####

interface=wlp5s0
bridge=br0
driver=nl80211

##### IEEE 802.11 related configuration #####

ssid=dd-wrt
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
wpa_passphrase=00112233445566778899AABBCC
```


https://wireless.wiki.kernel.org/en/developers/regulatory/statement



