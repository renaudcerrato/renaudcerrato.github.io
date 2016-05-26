---
published: false
layout: post
---

1-enable source in etc/apt/sources.list

UNAME=$(uname -r)
sudo apt-get build-dep linux-image-$UNAME

mkdir ~/tmp
cp /boot/config-$UNAME  ~/tmp/.config && cp /usr/src/linux-headers-${UNAME}/Module.symvers ~/tmp/

apt-get source linux-image-$UNAME


curl -L https://gist.github.com/renaudcerrato/ba9e200af202bb4f651fd2ba09adea6b/raw/ab36b11bb0c6357cc0513b2c6500a1841c8dd252/402-ath_regd_optional.patch | patch -p1 -b

make EXTRAVERSION=-${UNAME#*-} O=~/tmp oldconfig && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp prepare && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp modules_prepare && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp SUBDIRS=scripts/mod && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp modules SUBDIRS=drivers/net/wireless/ath

cd ~/tmp && sudo find . -wholename *drivers/net/wireless/ath*.ko -exec install -b {} /lib/modules/${UNAME}/kernel/{} \;

sudo depmod -a

sudo iw reg set US 
cat /etc/default/crda




https://wireless.wiki.kernel.org/en/developers/regulatory/statement



