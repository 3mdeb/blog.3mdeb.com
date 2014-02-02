---
layout: post
title: "Running U-boot on emulated IGEPv2 (DM3830)"
date: 2014-01-30 00:50:07 +0100
comments: true
categories: vdb embedded
published: false
keywords: u-boot, embedded, igepv2, dm3830
description: 
---

<p class=intro> First, you can ask why I choose IGEPv2 for this article ? And I
have to admit that I was strongly influenced by Free Electrons training
materials that I studied to improve my embedded skills. </p>

## Initial triage ##

When I take a close look at
[IGEPv2](https://www.isee.biz/products/igep-processor-boards/igepv2-dm3730)
site I found interesting [IGEP QEMU
Emulator](https://www.isee.biz/support/downloads/item/qemu-emulator) package.
As a embedded beginner I'm trying to be frugal and when I found QEMU I treat
it as an opportunity to save 180 EUR for a while and improve my skills in 
configuring virtual development environment.

Package contain few interesting things inside:

1. IGEP QEMU patch
2. igep-nano 512MB image
3. few qemu binaries (`qemu-ga`, `qemu-io,` `qemu-nbd,` ...)

To start ASAP lets investigate our image with [`binwalk`]():
```
[14:44:10] pkrol:igepv2-dm3730 git:(master*) $ tar jxvf QEMU-linaro.2012.12.01.tar.bz2
QEMU/
QEMU/0001-IGEP_QEMU_support.path
QEMU/qemu-linaro.tar.gz
QEMU/igep-nano.img.tgz
[14:47:39] pkrol:igepv2-dm3730 git:(master*) $ binwalk igep-nano.img

DECIMAL   	HEX       	DESCRIPTION
-------------------------------------------------------------------------------------------------------------------
1073440   	0x106120  	U-Boot boot loader reference
1170815   	0x11DD7F  	U-Boot boot loader reference
(...)
54525952  	0x3400000 	Linux EXT filesystem, rev 1.0 ext4 filesystem data, UUID=0008119d-9b57-4eb8-9a77-d16828b728b7, volume name "rootfs"
59163909  	0x386C505 	Linux kernel version "2.6.37+ (mcaro@manel-VirtualBox2) (gcc version 4.6.1 (Ubuntu/Li2) (gcc version 4.6.1 (Ubuntu/Linaro 4.6.1-7ubuntu2~ppa3) ) #1 "
(...)
```

From binwalk log we can read a lot of interesting things for example that we
have installed wpasupplicant, filesystem used is ext4 and kernel version is
greater than 2.6.37. So we have some disk image but how to connect it ? Quick
look into patch delivered in package show that we can use `-sd` or `-mtdblock`
parameter of Qemu.

I tried to run this image with attached `qemu-system-arm` and latest
`qemu-linaro` compiled from source. Support for IGEPv2 is not in qemu-linaro
upstream version, so I had to port attached patch. Unfortunately results for
both qemu versions are the same:

```
[18:03:20] pkrol:igepv2-dm3730 git:(master*) $ qemu-system-arm -M igep -sd igep-nano.img --serial stdio
VNC server running on `::1:5900'


IGEP-X-Loader 2.3.0-1 (Jan 10 2012 - 13:08:47)
XLoader: IGEPv2 : kernel boot ...
Uncompressing Linux... done, booting the kernel.
```

In this place emulation hanging forever.

NOTE: When I tried to run attached qemu on my x86_64 laptop I get some library
errors, like this below:

```
qemu-linaro/bin/qemu-system-arm: error while loading shared libraries: libbluetooth.so.3: cannot open shared object file: No such file or directory
```

to fix that you have to install i386 packages with the library i.e. `sudo
apt-get install libbluetooth3:i386`

## Ivestigation ##

So, I'm stuck for a while. My concerns in this point are:

* Kernel was configured for displaying messages on attached display, serial
  logging was disabled and Qemu VNC session doesn't show anything. I could
  split qemu-nano image into components and try to add some options to bootloader but it
  seems to be a hacky workaround.
* I will attach to Qemu session with crossdebugger and see whats going on I
  hope that kernel wan't stripped from debugging symbols.
* Maybe I should concentrate on building completely new kernel for this board
  with enabled debugging ?

### Debugging sessions ###

After building `arm-unknown-linux-uclibcgnueabi` toolchain using crosstool-ng I
tried to debug whats going on inside emulated environment:



## Qemu from Linaro ##

Unfortunately Qemu upstream doesn't contain
[IGEPv2](https://www.isee.biz/products/igep-processor-boards/igepv2-dm3730)
support 
