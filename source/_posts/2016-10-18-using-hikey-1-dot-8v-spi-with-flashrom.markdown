---
author: Piotr Król
layout: post
title: "Using HiKey 1.8V SPI with flashrom"
date: 2016-10-18 17:43:41 +0200
comments: true
categories: embedded hikey coreboot
---

Recently one of my customer asked me about evaluating effort for porting
coreboot to some platform. Key problem was that this platform used 1.8V SPI.
The only platform that I had with this voltage level was LeMaker HiKey.
Unfortunately default Linaro images do not expose SPI as device, what is
required by flashrom.

## Compile flashrom

```
sudo apt-get install subversion libpci-dev libusb-dev libusb-1.0.0-dev
svn co svn://flashrom.org/flashrom/trunk flashrom
cd flashrom
make
```

## Compile kernel with spi modules

Mainline kernel 4.8.1 was used:

```
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.8.1.tar.xz
tar xf linux-4.8.1.tar.xz
```

Obtain `linaro` toolchain:

```
http://releases.linaro.org/components/toolchain/gcc-linaro/4.9-2016.02/\
gcc-linaro-4.9-2016.02.tar.xz
mkdir ~/arm64-tc
tar -C ~/arm64-tc  gcc-linaro-4.9-2016.02.tar.xz
```

Add

Compile kernel

```
export PATH=~/arm64-tc/bin:$PATH
cd linux-4.8.1

```

flashrom:


sudo apt-get update

sudo apt-get install libpci-dev zlib1g-dev libftdi-dev libusb-dev libusb-1.0 \
                libusb-1.0-0-dev
```

download and unpack flashrom source:

```
http://download.flashrom.org/releases/flashrom-0.9.9.tar.bz2
tar xf flashrom-0.9.9.tar.bz2
```

build and install:

```
make
sudo make install
```


10517  ls
10518  sudo mount /dev/sdc1 /mnt
10519  ls /mnt
10520  sudo cp Image /mnt
10521  sudo cp dts/hisilicon/hi6220-hikey.dtb /mnt
10522  sudo umount /dev/sdc*
10523  sudo mount /dev/sdc2 /mnt
10524  sudo make ARCH=arm64 INSTALL_MOD_PATH=/mnt modules_install
10525  cd ../../../
10526  sudo make ARCH=arm64 INSTALL_MOD_PATH=/mnt modules_install


## Useful documenation

* [Hi6220V100 datasheet](http://mirror.lemaker.org/Hi6220V100_Multi-Mode_Application_Processor_Function_Description.pdf)
* [ARN PLL022](http://infocenter.arm.com/help/topic/com.arm.doc.ddi0194g/DDI0194G_ssp_pl022_r1p3_trm.pdf)
* [HiKey: Build Debian from source](http://wiki.lemaker.org/HiKey(LeMaker_version):Building_Debian_from_Source_Code)
