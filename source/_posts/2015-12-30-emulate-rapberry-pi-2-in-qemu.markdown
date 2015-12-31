---
layout: post
title: "Emulate Rapberry Pi 2 in QEMU"
date: 2015-12-30 23:02:30 +0100
comments: true
categories: qemu embedded linux
---

During planning system test for one of my projects I found that someone from
Microsoft published patches with [BCM2846 support](https://lists.gnu.org/archive/html/qemu-arm/2015-12/msg00078.html) to
QEMU mailing list. 

Those changes and 

## Get QEMU and compile

```
git clone https://github.com/0xabu/qemu.git -b raspi
git submodule update --init dtc
./configure
make -j$(nproc)
sudo make install
```

## Get recent Raspbian

```
wget http://downloads.raspberrypi.org/raspbian/images/raspbian-2015-11-24/2015-11-21-raspbian-jessie.zip
unzip 2015-11-21-raspbian-jessie.zip
[23:35:23] pietrushnic:rpi2_qemu $ sudo /sbin/fdisk -lu 2015-11-21-raspbian-jessie.img 
Disk 2015-11-21-raspbian-jessie.img: 3.7 GiB, 3934257152 bytes, 7684096 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xea0e7380

Device                          Boot  Start     End Sectors  Size Id Type
2015-11-21-raspbian-jessie.img1        8192  131071  122880   60M  c W95 FAT32 (LBA)
2015-11-21-raspbian-jessie.img2      131072 7684095 7553024  3.6G 83 Linux
```

Check start of `W95 FAT32 (LBA)` partition. It is `8192`. Sector size is `512`.
So calculate offset in bytes `8192 * 63 = 4194304`.

```
mkdir tmp
sudo mount -o loop,offset=4194304 2015-11-21-raspbian-jessie.img tmp
mkdir 2015-11-21-raspbian-boot
cp tmp/kernel7.img 2015-11-21-raspbian-boot
cp tmp/bcm2709-rpi-2-b.dtb 2015-11-21-raspbian-boot
```

TBD

## Debugging Raspbian in QEMU

```
qemu-system-arm -M raspi2 -kernel 2015-11-21-raspbian-boot/kernel7.img -sd \
2015-11-21-raspbian-jessie.img -append "rw earlyprintk loglevel=8 \
console=ttyAMA0,115200 dwc_otg.lpm_enable=0 root=/dev/mmcblk0p2" -dtb \
2015-11-21-raspbian-boot/bcm2709-rpi-2-b.dtb -usbdevice mouse -usbdevice \
keyboard -serial stdio -s -S
```
