---
published: false
layout: post
---

1-enable source in etc/apt/sources.list

## Configuration

VERSION=$(uname -r)
sudo apt-get build-dep linux-image-$VERSION

mkdir ~/build
cp /boot/config-$VERSION  ~/build/.config && cp /usr/src/linux-headers-${VERSION}/Module.symvers ~/build/

## Compilation

apt-get source linux-image-$VERSION

cd linux-${VERSION%%-*}

curl -L https://gist.github.com/renaudcerrato/ba9e200af202bb4f651fd2ba09adea6b/raw/ab36b11bb0c6357cc0513b2c6500a1841c8dd252/402-ath_regd_optional.patch | patch -p1 -b

make EXTRAVERSION=-${VERSION#*-} O=~/build oldconfig && \
make EXTRAVERSION=-${VERSION#*-} O=~/build prepare && \
make EXTRAVERSION=-${VERSION#*-} O=~/build modules_prepare && \
make EXTRAVERSION=-${VERSION#*-} O=~/build SUBDIRS=scripts/mod && \
make EXTRAVERSION=-${VERSION#*-} O=~/build modules SUBDIRS=drivers/net/wireless/ath

## Installation

cd ~/build && sudo find . -wholename *drivers/net/wireless/ath*.ko -exec install -b {} /lib/modules/${VERSION}/kernel/{} \; && sudo depmod -a


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



