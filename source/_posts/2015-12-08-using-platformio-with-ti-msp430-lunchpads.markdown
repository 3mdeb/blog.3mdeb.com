---
layout: post
title: "Using PlatformIO with TI MSP430 LunchPads"
date: 2015-12-08 13:16:36 +0100
comments: true
categories: msp430 embedded ti platformio
---

[PlatformIO](http://platformio.org/) is very interesting project that aim to
solve very important problem of configuring development environment for
embedded systems. Recent years we have explosion of bootstraping applications
(ie.vagrant, puppet). Most of them seems to follow git-like command line
interface and getting a lot of attention from programmers community. PlatformIO
is promising project for all Embedded Software developers who in the era of IoT
came from Linux systems.

It take some time to try PlatformIO using real hardware. Luckily on my desk
there are 2 supported boards gathering dust, which I would like to try in this
post.

## Installation on Debian

I highly recommend using
[virtualenv](https://virtualenv.readthedocs.org/en/latest/) for any custom
python application.

At the beginning I simply follow [Getting Started](http://platformio.org/#!/get-started) page.

```
pip install -U pip setuptools
pip install -U platformio
```

My 2 boards that I would like to try are: `MSP-EXP430FR5969` and `MSP-EXP430F5529LP`
First question is what configuration I should use ?

```
[13:38:34] pietrushnic:msp430 $ platformio boards|grep 5969
lpmsp430fr5969        msp430fr5969   8Mhz      64Kb    1Kb    TI LaunchPad w/ msp430fr5969
[13:38:41] pietrushnic:msp430 $ platformio boards|grep 5529
lpmsp430f5529         msp430f5529    16Mhz     128Kb   1Kb    TI LaunchPad w/ msp430f5529 (16MHz)
lpmsp430f5529_25      msp430f5529    25Mhz     128Kb   1Kb    TI LaunchPad w/ msp430f5529 (25MHz)
```

So it looks like `5529` have 2 flavours. It looks like they just differ in CPU
frequency configuration. 16MHz option is for backward compatibility. Let's use
more powerful 25MHz config.

```
mkdir msp430
cd msp430
platformio init --board=lpmsp430fr5969 --board=lpmsp430f5529_25
```

First question is about auto-uploading successfully built project, so I
answered y. Then PlatformIO inform about creating some directories and
`platformio.ini` file. After confirming toolchain downloading starts.

## Problems

### Lack of main.cpp

If you run platformio without any source code in `src` directory you will get error message like this:

```
.pioenvs/lpmsp430f5529_25/libFrameworkEnergia.a(main.o): In function `main':
/home/pietrushnic/projects/3mdeb/msp430/.pioenvs/lpmsp430f5529_25/FrameworkEnergia/main.cpp:7: undefined reference to `setup'
/home/pietrushnic/projects/3mdeb/msp430/.pioenvs/lpmsp430f5529_25/FrameworkEnergia/main.cpp:10: undefined reference to `loop'
collect2: ld returned 1 exit status
scons: *** [.pioenvs/lpmsp430f5529_25/firmware.elf] Error 1
```

Of course adding main.cpp to src directory fix this issue.

### libmsp430.so: cannot open shared object file

Next problem is with `libmsp430.so` which is not visible by `mspdebug`, but was
installed by platformio in
`$HOME/.platformio/packages/toolchain-timsp430/bin/libmsp430.so`.

Running:

```
export LD_LIBRARY_PATH=$HOME/.platformio/packages/toolchain-timsp430/bin/
```

before `platformio` fix problem.

### tilib: device initialization failed

If you didn't use your MSP430 for a while there can be problem like this:

```
/home/pietrushnic/.platformio/packages/tool-mspdebug/mspdebug tilib --force-reset "prog .pioenvs/lpmsp430f5529_25/firmware.hex"
MSPDebug version 0.20 - debugging tool for MSP430 MCUs
Copyright (C) 2009-2012 Daniel Beer <dlbeer@gmail.com>
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

MSP430_GetNumberOfUsbIfs
MSP430_GetNameOfUsbIf
Found FET: ttyACM0
MSP430_Initialize: ttyACM0
FET firmware update is required.
Re-run with --allow-fw-update to perform a firmware update.
tilib: device initialization failed
scons: *** [upload] Error 255
```

Fix for that according to error log should be like this:

```

```
