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

cd linux-${UNAME%%-*}
curl https://gist.github.com/renaudcerrato/bc943d8b5fe42622d533ea3de929c488/raw/d017fcaf85cec07c17d5d0cc9201fae3669cbf1b/402-ath_regd_optional.patch | patch -p1 -b

make EXTRAVERSION=-${UNAME#*-} O=~/tmp oldconfig && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp prepare && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp modules_prepare && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp SUBDIRS=scripts/mod && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp modules SUBDIRS=drivers/net/wireless/ath

sudo cp ~/tmp/drivers/net/wireless/ath/ath.ko /lib/modules/${UNAME}/kernel/drivers/net/wireless/ath/

sudo depmod -a

sudo iw reg set US 
cat /etc/default/crda




https://wireless.wiki.kernel.org/en/developers/regulatory/statement



