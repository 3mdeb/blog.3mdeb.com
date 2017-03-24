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

## Debugging problems

To debug Zephyr I used tui mode of gdb:

```
OPENOCD=/usr/local/bin/openocd TUI="--tui" make BOARD=frdm_k64f debug
```

Please note that before debugging you have to flash application to your target.
At least I saw that need when switching applications.

Unfortunately debugging didn't worked for me out of the box. I struggle with
various problems trying different configuration. My main goal was to have pure
OpenOCD+GDB environment. It happen very problematic with breakpoints triggering
exception handlers and GDB initially stopping in weird location.

I [asked](https://lists.zephyrproject.org/pipermail/zephyr-devel/2017-March/007352.html)
on mailing list question about narrowing down this issue. Moving forward with
limited debugging functionality would be harder, but not impossible - `print is your friend`.

NXP support replies on mailing list of course are far from being satisfying.

## Digging in OpenOCD

In general there were two issues I faced:

```
Error: 123323 44739 target.c:2898 target_wait_state(): timed out (>40000) while waiting for target halted
(...)
Error: 123917 44934 armv7m.c:723 armv7m_checksum_memory(): error executing cortex_m crc algorithm (retval=-302)
```

`timeout` value and `retval` value were added for debugging purposes. First
conclusion was that increasing timeout doesn't help and that crc failure could
be caused by problems with issuing halt, so it sound like both problems were
connected.

### DAPLink

It looks like [DAPLink](https://github.com/mbedmicro/DAPLink) replace mbed
CMSIS-DAP, but there is no clear information about support in OpenOCD except
that `pyOCD` should debug target with this firmware. Unfortunately DAPLink
firmware provided by NXP for FRDM-K64F didn't worked for me out of the box,
what I tried to resolve [here](https://community.nxp.com/thread/447692).

## Kinetis Design Studio

I don't like idea of bloated Eclipse-based IDEs forced on us by corpo-minds. It
looks like all of big semiconductors in embedded go that way TI, STM, NXP -
this terrible for industry and lot of Linux enthusiast working on IoT
solutions. Not mention Atmel, which is even worst going Visual Studio path and
making whole ecosystem terrible to work with.

I know they want to attract junior developers with good looking interface, but
number of option hidden and quality of documentation lead experts to rebel
against this choice.

Luckily KDS is available in DEB package, but it couldn't be smaller then 691MB.
No I have to allow this big bugged environment to dig into my system :(.

```
[1:31:54] pietrushnic:Downloads $ sudo dpkg -i  kinetis-design-studio_3.2.0-1_amd64.deb 
[sudo] password for pietrushnic:
Selecting previously unselected package kinetis-design-studio.
(Reading database ... 405039 files and directories currently installed.)
Preparing to unpack kinetis-design-studio_3.2.0-1_amd64.deb ...
Unpacking kinetis-design-studio (3.2.0) ...
Setting up kinetis-design-studio (3.2.0) ...

**********************************************************************
* Warning: This package includes the GCC ARM Embedded toolchain,     *
*          which is built for 32-bit hosts. If you are using a       *
*          64-bit system, you may need to install additional         *
*          packages before building software with these tools.       *
*                                                                    *
*          For more details see:                                     *
*          - KDS_Users_Guide.pdf:"Installing Kinetis Design Studio". *
*          - The Kinetis Design Studio release notes.                *
**********************************************************************
Processing triggers for gnome-menus (3.13.3-9) ...
Processing triggers for desktop-file-utils (0.23-1) ...
Processing triggers for mime-support (3.60) ...
```

Then this:

<picture>


### KDS OpenOCD

Interestingly OpenOCD in KDS behave composedly different. There are still
problem with errors mentioned above. Unfortunately flashing is terribly slow
(0.900 KiB/s). NXP seems to use old OpenOCD `Open On-Chip Debugger 0.8.0-dev (2015-01-09-16:23)`

