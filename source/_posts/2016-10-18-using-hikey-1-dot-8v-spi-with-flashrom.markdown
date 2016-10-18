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

## Useful documenation

* [Hi6220V100 datasheet](http://mirror.lemaker.org/Hi6220V100_Multi-Mode_Application_Processor_Function_Description.pdf)
* [ARN PLL022](http://infocenter.arm.com/help/topic/com.arm.doc.ddi0194g/DDI0194G_ssp_pl022_r1p3_trm.pdf)
