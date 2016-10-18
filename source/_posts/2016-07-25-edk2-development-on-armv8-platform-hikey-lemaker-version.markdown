---
author: Piotr Król
layout: post
title: "EDK2 development on ARMv8 platform (HiKey LeMaker version)"
date: 2016-07-25 14:13:02 +0200
comments: true
categories: uefi edk2 embedded armv8
---

This is second post from series about LeMaker version of HiKey board from
96boards Customer Edition family. [Previous](2016/05/19/powering-on-96boards-compatible-lemaker-hikey-armv8-for-uefi-development/)
post focused on describing hardware part. In this post I would like to show how
to setup firmware development and testing environment.

This post highly rely on [96boards documentation](https://github.com/96boards/documentation/wiki/HiKeyUEFI),
so kudos to 96boards and LeMaker for providing lot of information for developers.

## Obtain pre-compiled binaries

```
wget https://builds.96boards.org/snapshots/hikey/linaro/uefi/latest/l-loader.bin
wget https://builds.96boards.org/snapshots/hikey/linaro/uefi/latest/fip.bin
wget https://builds.96boards.org/snapshots/hikey/linaro/uefi/latest/ptable-linux-8g.img
wget https://builds.96boards.org/snapshots/hikey/linaro/uefi/latest/nvme.img
wget https://builds.96boards.org/releases/hikey/linaro/debian/latest/boot-fat.uefi.img.gz
wget http://builds.96boards.org/snapshots/hikey/linaro/debian/latest/hikey-jessie_developer_20160225-410.emmc.img.gz
gunzip *.img.gz
```

Clone eMMC flashing tool:

```
git clone https://github.com/96boards/burn-boot.git
```

Follow [flashing instructions](https://github.com/96boards/documentation/wiki/HiKeyUEFI#flash-binaries-to-emmc-).
For Debian-based systems you may need:

```
sudo apt-get install python-serial android-tools-fastboot
```

On my Debian I see in `dmesg`:

```
[21174.122832] usb 3-2.2: USB disconnect, device number 15
[21343.166870] usb 3-2.1.1: new full-speed USB device number 17 using xhci_hcd
[21343.268348] usb 3-2.1.1: New USB device found, idVendor=12d1, idProduct=3609
[21343.268352] usb 3-2.1.1: New USB device strings: Mfr=1, Product=4, SerialNumber=0
[21343.268353] usb 3-2.1.1: Product: \xffffffe3\xffffff84\xffffffb0㌲㔴㜶㤸
[21343.268355] usb 3-2.1.1: Manufacturer: 䕇䕎䥎
[21343.269159] option 3-2.1.1:1.0: GSM modem (1-port) converter detected
[21343.269271] usb 3-2.1.1: GSM modem (1-port) converter now attached to ttyUSB2
```

Correct command and UART log should look similar to this:

```
[17:11:36] pietrushnic:images $ sudo python ../src/burn-boot/hisi-idt.py --img1=l-loader.bin -d /dev/ttyUSB2
+----------------------+
(' Serial: ', '/dev/ttyUSB2')
(' Image1: ', 'l-loader.bin')
(' Image2: ', '')
+----------------------+

('Sending', 'l-loader.bin', '...')
Done
```

```
usb reset intr
reset device done.
start enum.
enum done intr
Enum is starting.
usb reset intr
enum done intr
NULL package
NULL package
USB ENUM OK.
init ser device done....
USB:: Err!! Unknown USB setup packet!
NULL package
USB:: Err!! Unknown USB setup packet!
NULL package
USB:: Err!! Unknown USB setup packet!
NULL package
USB:: Err!! Unknown USB setup packet!
NULL package
uFileAddress=ss=f9800800
uFileAddress=ss=f9800800

Switch to aarch64 mode. CPU0 executes at 0xf9801000!
```

As result I saw that green LED on board is on, then I proceed with fastboot
commands.

If above steps finish without the problems, then you know working procedure for
flashing all required components. Now let's proceed with fast boot and flashing
remaining components:

```
sudo fastboot flash ptable ptable-linux-8g.img
sudo fastboot flash fastboot fip.bin
sudo fastboot flash nvme nvme.img
sudo fastboot flash boot boot-fat.uefi.img
sudo fastboot flash system hikey-jessie_developer_20160225-410.emmc.img
```

Output should look like this:

```
$ sudo fastboot flash ptable ptable-linux-8g.img 
target reported max download size of 268435456 bytes
sending 'ptable' (17 KB)...
OKAY [  0.001s]
writing 'ptable'...
OKAY [  0.004s]
finished. total time: 0.006s
$ sudo fastboot flash fastboot fip.bin
target reported max download size of 268435456 bytes
sending 'fastboot' (1383 KB)...
OKAY [  0.060s]
writing 'fastboot'...
OKAY [  0.135s]
finished. total time: 0.196s
$ sudo fastboot flash nvme nvme.img
target reported max download size of 268435456 bytes
sending 'nvme' (128 KB)...
OKAY [  0.006s]
writing 'nvme'...
OKAY [  0.007s]
finished. total time: 0.014s
$ sudo fastboot flash boot boot-fat.uefi.img
target reported max download size of 268435456 bytes
sending 'boot' (65536 KB)...
OKAY [  2.645s]
writing 'boot'...
OKAY [  3.258s]
finished. total time: 5.903s
$ sudo fastboot flash system hikey-jessie_developer_20160225-410.emmc.img
target reported max download size of 268435456 bytes
sending sparse 'system' (262140 KB)...
OKAY [ 10.692s]
writing 'system'...
OKAY [ 11.868s]
sending sparse 'system' (262140 KB)...
OKAY [ 10.786s]
writing 'system'...
OKAY [ 11.838s]
sending sparse 'system' (262140 KB)...
OKAY [ 10.791s]
writing 'system'...
OKAY [ 11.812s]
sending sparse 'system' (262140 KB)...
OKAY [ 10.720s]
writing 'system'...
OKAY [ 11.803s]
sending sparse 'system' (262140 KB)...
OKAY [ 10.833s]
writing 'system'...
OKAY [ 11.830s]
sending sparse 'system' (116064 KB)...
OKAY [  4.854s]
writing 'system'...
OKAY [  5.219s]
finished. total time: 123.047s
```

Remove Boot Select jumper (link 3-4) and power on platform.

### System configuration

Wireless network can be easily configured using [this instructions](https://github.com/96boards/documentation/wiki/HiKeyGettingStarted#wireless-network).
It is also required to setup DNS in `/etc/resolv.conf` ie.:

```
nameserver 8.8.8.8
```

### Bug hunting

There was time when I asked myself what I can do ? Where to start ? Good way to
analyze system compatibility (and find bugs) from firmware perspective is
[FirmwareTestSuit](https://wiki.ubuntu.com/FirmwareTestSuite/). It can be
cloned using:

```
git clone git://kernel.ubuntu.com/hwe/fwts.git
```

To compile:

```
apt-get update
apt-get install autoconf automake libglib2.0-dev libtool libpcre3-dev libjson0-dev flex bison dkms
autoreconf -ivf
./configure
make
```

To run:

```
./src/fwts
```

At point of writing this post only 13 tests passed. Most of testes (243) were
aborted since no support for given feature was detected. This results show that
there is plenty to do before getting well-supported firmware on HiKey.

## Summary

