---
layout: post
title: "EDK2 development on ARMv8 platform (HiKey LeMaker version)"
date: 2016-07-25 14:13:02 +0200
comments: true
categories: uefi edk2 embedded armv8
---

This is second post from series about LeMaker version of HiKey board from
96boards Customer Edition family. [Previous](2016/05/19/powering-on-96boards-compatible-lemaker-hikey-armv8-for-uefi-development/)
post focused on describing hardware part. In this post I would like to show how
to start UEFI/EDK2 development on ARMv8 platform.

This post highly rely on [96boards documentation](https://github.com/96boards/documentation/wiki/HiKeyUEFI),
so kudos to 96boards for providing lot of information for developers.

## Obtain pre-compiled binaries

```
wget https://builds.96boards.org/releases/hikey/linaro/binaries/latest/l-loader.bin
wget https://builds.96boards.org/releases/hikey/linaro/binaries/latest/fip.bin
wget https://builds.96boards.org/releases/hikey/linaro/binaries/latest/ptable-linux-4g.img
wget https://builds.96boards.org/releases/hikey/linaro/debian/latest/boot-fat.uefi.img.gz
wget http://builds.96boards.org/releases/hikey/linaro/debian/latest/hikey-jessie_developer_20151130-387-4g.emmc.img.gz
gunzip *.img.gz
```

Clone eMMC flashing tool:

```
git clone https://github.com/96boards/burn-boot.git
```

Follow [flashing instructions](https://github.com/96boards/documentation/wiki/HiKeyUEFI#flash-binaries-to-emmc-).

For Debian-based systems you may need:

```
sudo apt-get install python-serial 
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

