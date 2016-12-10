---
author: Piotr Król
layout: post
title: "CryptoAuth OpenSSL engine and ECC508A"
date: 2016-11-27 23:20:31 +0100
comments: true
categories: openssl ecc508a security iot embedded linux
---

After [failure of my previous approach](2016/11/24/initial-triage-of-atmel-sam-g55-and-ecc508a/) I decide to
switch to something that seems to be more feasible. As project objectives states:

> Customers may buy un-programmed ATECC508A devices, download this project, build it, and establish a TLS1.2 connection without writing any code.
> Customers that buy personalized devices should be able to use these devices without writing any code.

Of course repository is getting older and no update was provided since February 2016. 

There were just 4 contributors where Atmel employee plus one consultant seemed
to provide most of the work. One closed issue, no merge requests no valuable
forks and 13 stars. I would not say this is vibrant community.

My goal was to confirm if project objectives are true and implement AWS Zero
Touch provisioning for Embedded Linux system. Method which utilize ECC508A was
overhyped by embedded news media in August 2016 what I described in [previous
post](2016/11/24/initial-triage-of-atmel-sam-g55-and-ecc508a/).

## Cryptoauth Xplained Pro

Luckily Atmel website provide enough information to continue evaluation of
ECC508A. CryptoAuth Xplained Pro board is a small evaluation board. It has
standard 20pin Atmel header to be compatible with various development boards
from the same vendor.

TODO: photo

Pinout is decribed in [hardware user guide](http://www.atmel.com/Images/Atmel-8893-CryptoAuth-XPro-Hardware-UserGuide.pdf).
Most development board have i2c connection exposed. Wiring is very simple all
you need is connect Vcc, GND, SCL and SDA.

After connecting Cryptoauth Xplained Pro your distro should show difference on
one of i2c buses. In my case it was `/dev/i2c-1`:

```
pi@raspberrypi:~/cryptoauth-openssl-engine $ i2cdetect 1
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will probe file /dev/i2c-1.
I will probe address range 0x03-0x77.
Continue? [Y/n] 
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: 50 -- -- -- -- -- -- -- 58 -- -- -- -- -- -- -- 
60: -- -- -- -- 64 -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --
```

When tried to read not provisioned device I saw something like this:

```
pi@raspberrypi:~/cryptoauth-openssl-engine $ sudo i2cdump 2 0x50
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will probe file /dev/i2c-2, address 0x50, mode byte
Continue? [Y/n] 
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
(...)
f0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
pi@raspberrypi:~/cryptoauth-openssl-engine $ sudo i2cdump 2 0x58
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will probe file /dev/i2c-2, address 0x58, mode byte
Continue? [Y/n] 
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: XX XX XX 04 11 33 43 04 11 33 43 04 11 33 43 04    XXX??3C??3C??3C?
10: 11 33 43 04 11 33 43 04 11 33 43 04 11 33 43 04    ?3C??3C??3C??3C?
(...)
f0: 11 33 43 04 11 33 43 04 11 33 43 04 11 33 43 04    ?3C??3C??3C??3C?
pi@raspberrypi:~/cryptoauth-openssl-engine $ sudo i2cdump 2 0x64
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will probe file /dev/i2c-2, address 0x64, mode byte
Continue? [Y/n] 
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: XX XX XX c9 c9 XX XX c9 c9 XX XX c9 c9 XX XX c9    XXX??XX??XX??XX?
10: c9 XX XX c9 c9 XX XX c9 c9 XX XX c9 c9 XX XX c9    ?XX??XX??XX??XX?
(...)
f0: c9 XX XX c9 c9 XX XX c9 c9 XX XX c9 c9 XX XX c9    ?XX??XX??XX??XX?
```

Lines that repeat were replaced with `(...)`.

## CryptoAuth OpenSSL Engine

This library was created by Atmel to provide support for TLS v1.2 connection by
utilizing ECC508A crypto coprocessor. To skip cross-compilation hassle I decide
to compile natively on my Embedded Linux. If you use development board with
recent SoC this should not be problem. My platform was RaspberryPi 2 with
`2016-11-25-raspbian-jessie`. To prepare my distro I needed additional steps:

```
sudo raspi-config
```

Choose `Advanced Options -> I2C` and enable i2c interface.

```console
sudo apt-get update
sudo apt-get install git vim tmux
```

Clone and compile:

```console
git clone https://github.com/AtmelCSO/cryptoauth-openssl-engine.git
cd cryptoauth-openssl-engine
# this will mesaure compilation time and pipe whole output to build.log
time make |& tee build.log
```

This takes ~17min.

## Provisioning

Let's switch context to crypt device provisioning. As it can be found in logs,
by dumping registers of i2c devices, chips are not in operational state. 

Unfortunately Root and Signer module can be used only with Windows application.
I had to run it in virtual machine. Of course first thing that came to mind is
to sniff traffic that go through USB bus. To do that Wireshark and `usbmon`
module can be used.

### USB monitor setup

```
sudo modprobe usbmon
```

This should cause additional devices appear in `/dev` like `/dev/usbmon0`,
`/dev/usbmon1` etc. Numbers means bus that you can sniff. So lets identify bus:

```
$ lsusb|grep microchip -i
Bus 003 Device 010: ID 04d8:0f30 Microchip Technology, Inc.
```

Then running Wireshark on `/dev/usbmon3` should show you flying URBs. Analyzing
this traffic should help in developing `Atmel Secure Provisioning Utilies` and
`Atmel Secure Provisioning Server` for Linux and other platforms. Legal
aftermath should be taken into consideration, but AFAIK 2 main points for
reversing USB and TCP/IP communication between provisioning client, server and
USB Root/Signer module are:

* there is not Linux/Mac implementation of this application, so user is forced
  to use Windows
* exact communication is not described and it may be implemented in insecure
  manner (ie. it may contain communication with outside server), so reasonable
  researcher should check if solution sold is good for his/her use

### AT88CKECCROOT

Under Linux it identify itself as:

```
[14344.115498] usb 3-1.3.2: new full-speed USB device number 10 using ehci-pci
[14344.239913] usb 3-1.3.2: New USB device found, idVendor=04d8, idProduct=0f30
[14344.239915] usb 3-1.3.2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[14344.239916] usb 3-1.3.2: Product: AWS Root dongle
[14344.239917] usb 3-1.3.2: Manufacturer: Microchip
```

### Atmel sales magic

TODO: picture

When taking this apart it happen to be just
[AT88CK590](http://www.atmel.com/tools/AT88CK590.aspx), what is interesting is
that this `AT88CK590` cost 20 USD but `AT88CKECCROOT` that seems to provide
3x`AT88CK590` cost [149.95USD](http://www.atmel.com/tools/AT88CKECCROOT-SIGNER.aspx?tab=overview)
what's even more interesting is that I see no difference between
`AT88CKECCROOT` and `AT88CKECCSIGNER`, but further one cost 99.95USD. This looks like sales magick.


### Lack of support for Linux i2c device

CryptoAuth OpenSSL Engines published by Atmel covers only one case when
[AT88CK101](http://www.atmel.com/tools/AT88CK101.aspx) development kit is used.
This means that Linux i2c device HAL had to be written and in addition to lack
of full ECC508A without NDA it required relying only on already implemented
HALs. There are dozen of them so it is not big problem, but still additional
work to just evaluate module.

I did comparison of [CryptoAuthLib Firmware Library 20160108](http://www.atmel.com/images/Atmel-CryptoAuthLib-Firmware_20160108.zip)
and what was included in engine. I see very little difference like:

* Makefiles were added to alldirectories
* Bunch of HALs for non Linux targets (ie. i2c bitbang, samd21, sam4s, samv71,
  swi etc.)
* Documentation improvements and in code comments
* correctly handle SHA length in couple functions
* `atcab_write_pubkey` was added basic module
* `tcatls_get_sn` was added atcatls module
* lots of other minor changes
* diffstat: `83 files changed, 2133 insertions(+), 1342 deletions(-)`

I did fork and update CryptoAuth Library in my repository. You can find update
version
[here](https://github.com/3mdeb/cryptoauth-openssl-engine/tree/cryptoauthlib_20160108)>
