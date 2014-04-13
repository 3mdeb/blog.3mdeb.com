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

## U-boot for IGEPv2 ##

Without kernel sources for image provided in IGEP Qemu package debugging is not
very easy task. Especially because `qemu-nano.img` contain only `vmlinuz,`
which is the compressed kernel executable not suitable for debugging purposes.

To create U-Boot image for IGEPv2 we need file that will represent SD card when
attached to Qemu.

### Image ###

Create image:

```
dd if=/dev/zero of=igepv2.img bs=1M count=256
```

Prepare partition in it:

```
[21:24:54] pkrol:igepv2-dm3730 git:(master*) $ sudo fdisk igepv2.img
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0xfc985d8c.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

Command (m for help): n
Partition type:
 p   primary (0 primary, 0 extended, 4 free)
 e   extended
Select (default p): p
Partition number (1-4, default 1):
Using default value 1
First sector (2048-524287, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-524287, default 524287): +64M

Command (m for help): t
Selected partition 1
Hex code (type L to list codes): c
Changed system type of partition 1 to c (W95 FAT32 (LBA))

Command (m for help): a
Partition number (1-4): 1

Command (m for help): w
The partition table has been altered!


WARNING: If you have created or modified any DOS 6.x
partitions, please see the fdisk manual page for additional
information.
Syncing disks.
```

To format our new FAT32 partition we will map piece of `igepv2.img` file to
loop device. Note that first sector of FAT32 partition is on `2048\*512` byte.

```
sudo losetup -o$[2048*512] /dev/loop1 ./igepv2.img
sudo mkfs.vfat -n boot -F 32 /dev/loop1
sudo losetup -d /dev/loop1
```

### Compile U-Boot ###

This step is very simple. Key is in choosing config file. For our Qemu powered IGEPv2 it will be `igep0020_nand_config`.
So, correct steps are:

```
git clone git://git.denx.de/u-boot.git
cd u-boot
make igep0020_nand_config
make
```

After all we can copy `MLO` and `u-boot.img` to `igepv2.img`:

```
sudo mkdir /mnt/tmp
sudo mount -o offset=$[2048*512] /path/to/igepv2.img /mnt/tmp
sudo cp MLO /mnt/tmp
sudo cp u-boot.img /mnt/tmp
```

## Qemu from Linaro ##

Unfortunately Qemu upstream doesn't contain
[IGEPv2](https://www.isee.biz/products/igep-processor-boards/igepv2-dm3730)
support 
