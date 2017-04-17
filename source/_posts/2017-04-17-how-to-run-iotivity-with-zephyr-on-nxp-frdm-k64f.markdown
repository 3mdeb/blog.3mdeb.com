---
author: Piotr Król
layout: post
title: "How to run IoTivity with Zephyr on NXP FRDM-K64F"
date: 2017-04-17 21:45:24 +0200
comments: true
categories: iotivity zephyr rtos nxp frdm-k64f
---

As consulting company we know that each project evaluation should start with
checking and verifying what features are already provided in existing projects,
especially those with permissive licenses.

Recently we were asked to port IoTvity to yet unsupported operating system
dedicated to some specific platform. Precisely query was about
[IoTvity-Constrained](https://github.com/iotivity/iotivity-constrained).
IoTvity is reference implementation of [Open Interconnect Consortium specification](https://openconnectivity.org/resources/specifications), which now exist under
name Open Connectivity Foundation. Don't ask me why they mess so much with
names making it really hard to understand what is what.

This is our first touch of IoTvity, but at first glance I see couple things:

- most examples contain CoAP as communication protocol, which IMO associate
  with IPv6,
- total specification has almost 500 pages - reasonable economic output would
  be required to read through those documents,
- there are multiple APIs availabe: C, C++ and Java
- Apache 2.0 license
- feature set is blasting and far from KISS philosophy

But let's try what we can do IoTvity-constrained basic use case from
documentation.

## Trying IoTvity with Zephyr on QEMU

```
git clone https://github.com/iotivity/iotivity-constrained.git
git submodule update --init
```

I assume you followed Zephyr [Getting Started](https://www.zephyrproject.org/doc/getting_started/getting_started.html)
or read [my post about Zephyr](2017/03/18/development-environment-for-zephyros-on-nxp-frdm-k64f).

```
cd port/zephyr
source /path/to/zephyr-project/zephyr-env.sh
make pristine && make
```

We can check RAM and ROM consumption, by using `make ram_report` and `make
rom_report`. Depending on perspective IoTivity-constrained take quite a lot of
memory RAM: 26.3KB and ROM: 63.6KB. It means it requires at least Cortex-M3
device.

To test we need Zephyr `net-tools`:

```
git clone https://gerrit.zephyrproject.org/r/p/net-tools.git
cd net-tools
make
./loop-socat.sh # in 1st terminal
sudo ./loop-slip-tap.sh # in 2nd terminal
```

## Trying IoTvity with Zephyr on NXP FRDM-K64F

First you need `python3-yaml` because without that you can get:

```
(py2.7-venv) [0:40:39] pietrushnic:zephyr git:(master) $ make pristine && make BOARD=frdm_k64f
Using /home/pietrushnic/storage/wdc/projects/2017/3mdeb/zephyr-project/boards/arm/frdm_k64f/frdm_k64f_defconfig as base
Merging /home/pietrushnic/storage/wdc/projects/2017/3mdeb/zephyr-project/kernel/configs/kernel.config
Merging prj.conf
warning: (BLUETOOTH_DEBUG_LOG && NET_BUF_LOG && NET_BUF_SIMPLE_LOG && NET_LOG) selects SYS_LOG which has unmet direct dependencies (PRINTK)
#
# configuration written to .config
#
make[1]: Entering directory '/home/pietrushnic/storage/wdc/projects/2017/3mdeb/zephyr-project'
make[2]: Entering directory '/home/pietrushnic/storage/wdc/projects/2017/3mdeb/iotivity-k64f/iotivity-constrained/port/zephyr/outdir/frdm_k64f'
  GEN     ./Makefile
scripts/kconfig/conf --silentoldconfig Kconfig
warning: (BLUETOOTH_DEBUG_LOG && NET_BUF_LOG && NET_BUF_SIMPLE_LOG && NET_LOG) selects SYS_LOG which has unmet direct dependencies (PRINTK)
warning: (BLUETOOTH_DEBUG_LOG && NET_BUF_LOG && NET_BUF_SIMPLE_LOG && NET_LOG) selects SYS_LOG which has unmet direct dependencies (PRINTK)
  Using /home/pietrushnic/storage/wdc/projects/2017/3mdeb/zephyr-project as source for kernel
  GEN     ./Makefile
  CHK     include/generated/version.h
  UPD     include/generated/version.h
  CHK     misc/generated/configs.c
  UPD     misc/generated/configs.c
  DTC     dts/arm/frdm_k64f.dts_compiled
  CHK     include/generated/generated_dts_board.h
Traceback (most recent call last):
  File "/home/pietrushnic/storage/wdc/projects/2017/3mdeb/zephyr-project/scripts/extract_dts_includes.py", line 6, in <module>
    import yaml
ImportError: No module named 'yaml'
/home/pietrushnic/storage/wdc/projects/2017/3mdeb/zephyr-project/./Kbuild:139: recipe for target 'include/generated/generated_dts_board.h' failed
make[3]: *** [include/generated/generated_dts_board.h] Error 1
/home/pietrushnic/storage/wdc/projects/2017/3mdeb/zephyr-project/Makefile:981: recipe for target 'prepare' failed
make[2]: *** [prepare] Error 2
make[2]: Leaving directory '/home/pietrushnic/storage/wdc/projects/2017/3mdeb/iotivity-k64f/iotivity-constrained/port/zephyr/outdir/frdm_k64f'
Makefile:177: recipe for target 'sub-make' failed
make[1]: *** [sub-make] Error 2
make[1]: Leaving directory '/home/pietrushnic/storage/wdc/projects/2017/3mdeb/zephyr-project'
/home/pietrushnic/storage/wdc/projects/2017/3mdeb/zephyr-project/Makefile.inc:82: recipe for target 'all' failed
make: *** [all] Error 2
```

To fulfill requirements you can use:

```
sudo apt install python3-yaml
```


