---
layout: post
title: "Powering on 96boards compatible LeMaker HiKey (ARMv8) for UEFI development"
date: 2016-05-19 00:04:06 +0200
comments: true
categories: embedded arm uefi
---

A lot happen in embedded world these days. IoT noise poping up every corner,
every conference talk have this words on agenda. ARM expansion trying to touch
server market and UEFI coming to non-x86 platforms. Because firmware these days
is really important for small and big platforms and in my work I touch both IoT
Cortex-M3 and full featured microarchitectures like Cortex-A53 I decided to try
back to roots and run UEFI on ARMv8 board.


## Choosing ARMv8 dev board

First problem was to choose development board. Probably simpler solution is to
use platforms like [Raspberru Pi 3](https://www.raspberrypi.org/magpi/raspberry-pi-3-specs-benchmarks/) which
features Broadcom Cortex-A53 or very interesting alternative like
[PINE64](https://www.pine64.com/product#intro) with Allwinner flavour.

Of course rush on this market bring other players like Amlogic with
[Odroic-C2](http://www.hardkernel.com/main/products/prdt_info.php?g_code=G145457216438).
It is worth to mention that adaptation of new architecture is very slow. It was
announced in 2012. First real product was released by Apple (iPhone S5), but
since then not much appeared on low end development board market. Things start
to change last year.

I have RPi3 on my desk but playing with its low level side is unknown since
limitation Broadcom put on releasing any information about BCM2837. My goal was
to work on UEFI and [ARM trusted firmware](https://github.com/ARM-software/arm-trusted-firmware)
the only board except expensive ARM reference platforms that seems to work with
UEFI was LeMaker HiKey.

### Why 96boards ?

* this is open specification
* its driven by Linaro, which is in my opinion do a lot of great work for whole
  community
* its standardized way with big players behind, so knowing it and having in
  portfolio cannot demage Embedded Systems Consultant career


## 1.8V UART problem

I finally get my board, but to my surprise realized that it use 1.8V level for
UART. Cables for that level are built with FT230XS or similar chips, which cost
~3USD. To my suprise cable that work with 1.8V UART level cost 30USD. There are
2 separated UART pins to connect on HiKey. One for low level bootloader
development and one for Linux kernel development. So I would need to cables.
Board cost 75USD, so you paying almost the same price for cables. It was not
acceptable for me.

Linaro developers seems to use [this](http://www.seeedstudio.com/depot/96Boards-UART-p-2525.html)
which is out of stock for 5 months!

While searching for alternatives I found [this TI converter on SparkFun page](https://www.sparkfun.com/products/11771).
Luckily availability of various SparkFun distributors made delivery possible in
less then 48h.
