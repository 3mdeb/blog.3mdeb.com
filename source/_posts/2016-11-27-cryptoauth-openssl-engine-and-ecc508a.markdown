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
one of i2c buses. In my case it was `/dev/i2c-2`:

```
debian@linux:~$ sudo i2cdetect  2
[sudo] password for debian:
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will probe file /dev/i2c-2.
I will probe address range 0x03-0x77.
Continue? [Y/n]
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- 48 -- -- -- -- -- -- --
50: 50 -- -- -- -- -- -- -- 58 -- -- -- -- -- -- --
60: -- -- -- -- 64 -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

When tried to read not provisioned device I saw something like this:

```
debian@linux:~/cryptoauth-openssl-engine$ sudo i2cdump 2 0x50
No size specified (using byte-data access)
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will probe file /dev/i2c-2, address 0x50, mode byte
Continue? [Y/n] 
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
(...)
f0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
@linux:~/cryptoauth-openssl-engine$ sudo i2cdump 2 0x58
No size specified (using byte-data access)
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will probe file /dev/i2c-2, address 0x58, mode byte
Continue? [Y/n] 
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: XX XX XX 04 11 33 43 04 11 33 43 04 11 33 43 04    XXX??3C??3C??3C?
10: 11 33 43 04 11 33 43 04 11 33 43 04 11 33 43 04    ?3C??3C??3C??3C?
(...)
f0: 11 33 43 04 11 33 43 04 11 33 43 04 11 33 43 04    ?3C??3C??3C??3C?
debian@linux:~/cryptoauth-openssl-engine$ sudo i2cdump 2 0x64
No size specified (using byte-data access)
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
recent SoC this should not be problem. My platform was i.MX6DL based, so I had
decent power behind. As Embedded Linux I used Debian image provided by vendor,
but my target it implementation of below work in Yocto image.

To prepare my distro I needed additional steps:

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

This takes ~14min on i.MX6DL (2x ARM® Cortex™-A9 up to 1 GHz).

Unfortunately compilation failed on my platform with errors like this:

```console
/hashes' make[3]: Leaving directory '/home/debian/cryptoauth-openssl-engine/engine_atecc/cryptoauthlib/lib/crypto'
/bin/sh: 1: make[3]:: not found
/bin/sh: 1: make[4]:: not found
Makefile:18: recipe for target 'all' failed
make[3]: [all] Error 127 (ignored)
(...)
make[4]: Leaving directory '/home/debian/cryptoauth-openssl-engine/engine_atecc/cryptoauthlib/test/atcacert'
make[4]: Entering directory '/home/debian/cryptoauth-openssl-engine/engine_atecc/cryptoauthlib/test/tls'
gcc -I. -I.. -I../.. -I../../.. -I../../../.. -I../../lib -I../lib -fPIC -g -O0 -DATCA_HAL_KIT_CDC -DATCAPRINTF -o atcatls_tests.o -c atcatls_tests.c
make[4]: Leaving directory '/home/debian/cryptoauth-openssl-engine/engine_atecc/cryptoauthlib/test/tls'
make[4]: Entering directory '/home/debian/cryptoauth-openssl-engine/engine_atecc/cryptoauthlib/test/sha-byte-test-vectors'
make[4]: *** No targets specified and no makefile found.  Stop.
make[4]: Leaving directory '/home/debian/cryptoauth-openssl-engine/engine_atecc/cryptoauthlib/test/sha-byte-test-vectors'
../Makefile.generic:14: recipe for target 'all' failed
make[3]: *** [all] Error 2
make[3]: Leaving directory '/home/debian/cryptoauth-openssl-engine/engine_atecc/cryptoauthlib/test'
Makefile:22: recipe for target 'tgt_test' failed
make[2]: *** [tgt_test] Error 2
make[2]: Leaving directory '/home/debian/cryptoauth-openssl-engine/engine_atecc/cryptoauthlib'
Makefile:39: recipe for target 'tgt_cryptoauthlib' failed
make[1]: *** [tgt_cryptoauthlib] Error 2
make[1]: Leaving directory '/home/debian/cryptoauth-openssl-engine/engine_atecc'
Makefile:104: recipe for target 'build_engine_atecc' failed
make: *** [build_engine_atecc] Error 2
```

This error is not consistent and objects that had problem vary. On x86
everything compiles without errors so it have to be toolchain issue.

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
