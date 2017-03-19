---
author: Piotr Król
layout: post
title: "Development environment for Zephyr on NXP FRDM-K64F"
date: 2017-03-18 15:27:05 +0100
comments: true
categories: nxp frdm-k64f embedded zephyros iiot
---

In this post I would like to describe process of setting up NXP FRDM-K64F
development environment under Linux and start Zephyr development using it.

Why NXP FRDM-K64F ? I choose this platform mostly because of ready to use guide
about using 802.15.4 communication by attaching TI CC2520, which was presented
[here](https://wiki.zephyrproject.org/view/TI_CC2520#Use_case:_CC2520_on_NXP_FRDM-K64F),
because TI is little bit delayed with shipping CC2520 I decide to familiarize
myself with Zephyr networking stack and maybe try to implement some features.

One thing that came to my mind is 6LowPAN over Ethernet. 6LowPAN is very
popular in industrial internet of things, it is good to know tat protocol. From
materials that I found RIoT and Contiki RTOSes already have this features
included. It would be interesting to enable it for NXP FRDM-K64F and Zephyr.

## Setup

I started with initial triage if board works:

```
git clone https://gerrit.zephyrproject.org/r/zephyr && cd zephyr && git checkout tags/v1.7.0
cd zephyr
git checkout net
source zephyr-env.sh
cd $ZEPHYR_BASE/samples/hello_world/
make BOARD=frdm_k64f
cp outdir/frdm_k64f/zephyr.bin /media/pietrushnic/MBED/
```

On `/dev/ttyACM0` I get:

```
**** BOOTING ZEPHYR OS v1.7.99 - BUILD: Mar 18 2017 14:14:37 *****                                                                    |
Hello World! arm   
```

So it works great out of the box. Unfortunately  it is not possible to flash
using typical Zephyr OS command:

```
[15:16:13] pietrushnic:hello_world git:(k64f-ethernet) $ make BOARD=frdm_k64f flash
make[1]: Entering directory '/home/pietrushnic/storage/wdc/projects/2017/acme/src/zephyr'
make[2]: Entering directory '/home/pietrushnic/storage/wdc/projects/2017/acme/src/zephyr/samples/hello_world/outdir/frdm_k64f'
  Using /home/pietrushnic/storage/wdc/projects/2017/acme/src/zephyr as source for kernel
  GEN     ./Makefile
  CHK     include/generated/version.h
  CHK     misc/generated/configs.c
  CHK     include/generated/generated_dts_board.h
  CHK     include/generated/offsets.h
make[3]: 'isr_tables.o' is up to date.
Flashing frdm_k64f
Flashing Target Device
Open On-Chip Debugger 0.9.0-dirty (2016-08-02-16:04)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : only one transport option; autoselect 'swd'
Info : add flash_bank kinetis k60.flash
adapter speed: 1000 kHz
none separate
cortex_m reset_config sysresetreq
Error: unable to find CMSIS-DAP device

Done flashing
```

This statement is clear. Unfortunately this is not the whole solution for the problem.

NXP FRDM-K64F lacks correct firmware for debugger and that's why OpenOCD refuse
to cooperate. Recent CMSIS-DAP firmware can be installed by using [this guide](https://developer.mbed.org/handbook/Firmware-FRDM-K64F)
although it has limitation about which you can read [here](https://mcuoneclipse.com/2014/04/27/segger-j-link-firmware-for-opensdav2/).

Weird thing was that following [this post](http://jany.st/tag/frdm-k64f.html)
locally compiled OpenOCD worked with FRDM-K64F. I figure out that Zephyr use
OpenOCD from SDK by looking at logs see compilation date difference `Open On-Chip Debugger 0.9.0-dirty (2016-08-02-16:04)`.

Custom OpenOCD can be provided to Zephyr `make` system, by using `OPENOCD`
variable, so:

```
OPENOCD=/usr/local/bin/openocd make BOARD=frdm_k64f flash
OPENOCD=/usr/local/bin/openocd make BOARD=frdm_k64f debug
```

Both worked fine for me. I realized that I use `0.8.2` SDK, That why made
upgrade:

### Zephyr SDK upgrade

To get location of SDK:

```
$ source zephyr-env.sh 
$ echo $ZEPHYR_SDK_INSTALL_DIR
/home/pietrushnic/projects/2016/acme/zephyr_support/src/sdk
```

To upgrade:

```
$ wget https://nexus.zephyrproject.org/content/repositories/releases/org/zephyrproject/zephyr-sdk/0.9/zephyr-sdk-0.9-setup.run
$ chmod +x zephyr-sdk-0.9-setup.run 
$ ./zephyr-sdk-0.9-setup.run 
Verifying archive integrity... All good.
Uncompressing SDK for Zephyr  100% 
Enter target directory for SDK (default: /opt/zephyr-sdk/): /home/pietrushnic/projects/2016/acme/zephyr_support/src/sdk
Installing SDK to /home/pietrushnic/projects/2016/acme/zephyr_support/src/sdk
The directory /home/pietrushnic/projects/2016/acme/zephyr_support/src/sdk/sysroots will be removed!
Do you want to continue (y/n)?
y
 [*] Installing x86 tools...
 [*] Installing arm tools...
 [*] Installing arc tools...
 [*] Installing iamcu tools...
 [*] Installing nios2 tools...
 [*] Installing xtensa tools...
 [*] Installing riscv32 tools...
 [*] Installing additional host tools...
Success installing SDK. SDK is ready to be used.
```

Unfortunately recent SDK doesn't help with CMSIS-DAP problem, because of that I
switched to use upstream version of OpenOCD.

## zperf

`zperf` is network traffic generator included in sample applications of
Zephyr. It supports K64F, so to it was great place to start with networking.

```
cd $ZEPHYR_BASE/samples/net/zperf
OPENOCD=/usr/local/bin/openocd make BOARD=frdm_k64f flash
```

On terminal I saw:

```
zperf>
[zperf_init] Setting IP address 2001:db8::1
[zperf_init] Setting destination IP address 2001:db8::2
[zperf_init] Setting IP address 192.0.2.1
[zperf_init] Setting destination IP address 192.0.2.2
```

Testing scenarios are described [here](https://www.zephyrproject.org/doc/samples/net/zperf/README.html?highlight=zperf).
Unfortunately basic test hangs.

## Debugging

To debug Zephyr I used tui mode of gdb:

```
OPENOCD=/usr/local/bin/openocd TUI="--tui" make BOARD=frdm_k64f debug
```

Please note that before debugging you have to flash application to your target.
At least I saw that need when switching applications.






