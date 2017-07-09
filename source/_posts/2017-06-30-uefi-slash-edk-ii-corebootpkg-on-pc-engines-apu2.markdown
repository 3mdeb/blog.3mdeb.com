---
author: Piotr Król
layout: post
title: "UEFI/EDK II CorebootPkg on PC Engines APU2"
date: 2017-06-30 22:14:55 +0200
comments: true
categories: coreboot uefi edk2 apu2
---

Recently we were reached by person interested in running CoreOS on APU2. CoreOS
is very interesting system from security point of view it was created to
support containers out of the box.

But what is more interesting to firmware developer is idea of booting
UEFI-aware operating system no matter if it is CoreOS or Debian. Because of
that we decided to triage [coreboot UEFI payload](https://github.com/tianocore/tianocore.github.io/wiki/Coreboot_UEFI_payload)
on PC Engines APU2 platform.

For interested in that topi I recommend to take look at [video from coreboot conference 2016](https://youtu.be/I08NHJLu6Us?list=PLiWdJ1SEk1_AfMNC6nD_BvUVCIsHq6f0u).

All my modification from below article can be found in [3mdeb edk2 fork](https://github.com/3mdeb/edk2/tree/apu2-uefi)

## Firmware

Let's start with building APU2 mainline. First follow [this instruction](https://github.com/pcengines/release_manifests)
and build mainline version of coreboot. Meanwhile you can take care of EDK2 CorebootPkg build:

```
git clone https://github.com/tianocore/edk2.git
cd edk2
source edksetup.sh
```

On my Debian testing I had to explicitly change compilers to `gcc-5` and `g++-5`:

```
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 10
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-5 10
```

Otherwise build fails. After that I was able to build `UEFIPAYLOAD.fd`:

```
make -c BaseTools
build -a IA32 -p CorebootPayloadPkg/CorebootPayloadPkgIa32.dsc -b DEBUG -t GCC5
```

Build result is located in
`Build/CorebootPayloadPkgIA32/DEBUG_GCC5/FV/UEFIPAYLOAD.fd`. Following [build and integration instructions](https://raw.githubusercontent.com/tianocore/edk2/master/CorebootPayloadPkg/BuildAndIntegrationInstructions.txt)
I added build result as `An ELF executable payload`.

{image tbd}

It is important to deselect secondary payloads like `memtest86+` and
`sortbootorder` to avoid compilation issues.

### EDK2 development

For all interested in EDK2 code development I strongly advise to follow [Laszlo's  guide](https://github.com/tianocore/tianocore.github.io/wiki/Laszlo's-unkempt-git-guide-for-edk2-contributors-and-maintainers).

Useful script in case of APU2 may be:

```sh
#!/bin/bash
build -a IA32 -p CorebootPayloadPkg/CorebootPayloadPkgIa32.dsc -b DEBUG -t GCC5
cp Build/CorebootPayloadPkgIA32/DEBUG_GCC5/FV/UEFIPAYLOAD.fd ../apu2_fw_rel/apu2/coreboot/
cd ../apu2_fw_rel
./apu2/apu2-documentation/scripts/apu2_fw_rel.sh build-ml
read -n 1 -s -p "Press any key to continue with flashing recent build ..."
./apu2/apu2-documentation/scripts/apu2_fw_rel.sh flash-ml root@192.168.0.105
cd ../edk2
```

I added prompt for flashing since it happen to forget remove recovery flash,
what may lead to blocking further development since both SPIs will contain not
bootable firmware.


## Booting

### CbSupportDxe assert

First problem I faced was assert in `CbSupportDxe.c` related to adding 1MB
memory for LAPIC. Reservation happen in CbSupportDxe entry point.

```
Loading driver at 0x000CFC25000 EntryPoint=0x000CFC2595B CbSupportDxe.efi
InstallProtocolInterface: BC62157E-3E33-4FEC-9920-2D3B36D750DF CFC42190
ProtectUefiImageCommon - 0xCFC42928
  - 0x00000000CFC25000 - 0x0000000000002580
PROGRESS CODE: V03040002 I0
Failed to add memory space :0xFEE00000 0x100000

ASSERT_EFI_ERROR (Status = Access Denied)

DXE_ASSERT!: [CbSupportDxe] /home/pietrushnic/storage/wdc/projects/2017/pcengines/apu/src/edk2/CorebootModulePkg/CbSupportDxe/CbSupportDxe.c (57): !EFI_ERROR (Status)
```

Interestingly CbSupportDxe seems to not have problem with finding GUID HOB,
which is right after asserting reservation. It also pass installation of ACPI
and SMBIOS table. I just commented that reservation and moved forward to see
what will happen.

### PcRtcEntry assert

Next problem happen during initialization of RTC:

```
Loading driver at 0x000CFE50000 EntryPoint=0x000CFE507DB PcRtc.efi
InstallProtocolInterface: BC62157E-3E33-4FEC-9920-2D3B36D750DF CFC2AE90
ProtectUefiImageCommon - 0xCFC2AB28
  - 0x00000000CFE50000 - 0x0000000000003980
  PROGRESS CODE: V03040002 I0
  ASSERT_EFI_ERROR (Status = Device Error)

  DXE_ASSERT!: [PcRtc] /home/pietrushnic/storage/wdc/projects/2017/pcengines/apu/src/edk2/PcAtChipsetPkg/PcatRealTimeClockRuntimeDxe/PcRtcEntry.c (143): !EFI_ERROR (Status)
```

Unfortunately Real Time Clock is required architecture protocol and cannot be
omitted.

First problem with this code was incorrect state of `Valid RAM and Time` (`VRT`) bit in
RTC Date Alarm register (aka Register D).

Checking BKDG I was not able to find issue with RTC reading all registers was
done correctly and register layout seemed to be standardized for RTC devices.

I faced one very strange situation after leaving APU2 for a night. First boot
passed through above assert and finished booting much farther (in BDS).
This was very suspicious like timing or HW initialization issue. Final log
looked like that:

```
[Bds]=============Begin Load Options Dumping ...=============
  Driver Options:
  SysPrep Options:
  Boot Options:
    Boot0000: UiApp              0x0109
    Boot0001: UEFI Shell                 0x0001
  PlatformRecovery Options:
    PlatformRecovery0000: Default PlatformRecovery               0x0001
[Bds]=============End Load Options Dumping=============
[Bds]BdsWait ...Zzzzzzzzzzzz...
[Bds]BdsWait(3)..Zzzz...
[Bds]BdsWait(2)..Zzzz...
[Bds]BdsWait(1)..Zzzz...
[Bds]Exit the waiting!
PROGRESS CODE: V03051007 I0
[Bds]Stop Hotkey Service!
[Bds]UnregisterKeyNotify: 000C/0000 Success
[Bds]UnregisterKeyNotify: 0002/0000 Success
[Bds]UnregisterKeyNotify: 0000/000D Success
Enable SCI bit at 0x804 before boot
PROGRESS CODE: V03051001 I0
Memory  Previous  Current    Next
 Type    Pages     Pages     Pages
======  ========  ========  ========
  09    00000008  00000000  00000008
  0A    00000004  00000000  00000004
  00    00000004  00000001  00000004
  06    000000C0  0000002E  000000C0
  05    00000080  0000001A  00000080
[Bds]Booting UEFI Shell
[Bds] Expand MemoryMapped(0xB,0x830000,0xC0FFFF)/FvFile(C57AD6B7-0515-40A8-9D21-551652854E37) -> MemoryMapped(0xB,0x830000,0xC0FFFF)/FvFile(C57AD6B7-0515-40A8-9D21-551652854E37)
PROGRESS CODE: V03058000 I0
InstallProtocolInterface: 5B1B31A1-9562-11D2-8E3F-00A0C969723B CF954D28
Loading driver at 0x000CF6B8000 EntryPoint=0x000CF718BC1
InstallProtocolInterface: BC62157E-3E33-4FEC-9920-2D3B36D750DF CF96E590
ProtectUefiImageCommon - 0xCF954D28
  - 0x00000000CF6B8000 - 0x00000000000A6FA0
PROGRESS CODE: V03058001 I0
InstallProtocolInterface: 47C7B221-C42A-11D2-8E57-00A0C969723B CF6BCA38
InstallProtocolInterface: 47C7B223-C42A-11D2-8E57-00A0C969723B CF945410
```

In debug logs there was nothing suspicious. Apparently register D of RTC
returned correct value in `VRT` register.

Finally it happen that `VRT` was incorrectly described in datasheet as
read-only. Register D initialization function caused setting `VRT` bit to 0
what further led to `Device Error` assert. I fixed that problem by removing
initialization from `PcRtcInit`.

### Unexpected behavior

Other behaviors worth to note were unexpected coreboot reset after applying
power:

```
PCEngines apu2
coreboot build 06/30/2017
BIOS version v4.5.8
PCEngines apu2
coreboot build 06/30/2017
BIOS version v4.5.8
4080 MB ECC DRAM

PROGRESS CODE: V03020003 I0
Loading PEIM at 0x000008143C0 EntryPoint=0x00000814600 CbSupportPeim.efi
PROGRESS CODE: V03020002 I0
```

## Booting to UEFI Shell

I had freeze after:

```
InstallProtocolInterface: 47C7B221-C42A-11D2-8E57-00A0C969723B CF6BCA38
InstallProtocolInterface: 47C7B223-C42A-11D2-8E57-00A0C969723B CF945410
```

First is `gEfiShellEnvironment2Guid` and second `gEfiShellInterfaceGuid`, so I
decided to take a look where those GUIDs are used and hook there to see what
may be wrong. After poking around I realized that those came from binary
included in repository. What is included can be modified by changing
`SHELL_TYPE` variable.

When using `BUILD_SHELL` I see little bit different output:

```
InstallProtocolInterface: 387477C2-69C7-11D2-8E39-00A0C969723B CF8DEBA0
InstallProtocolInterface: 752F3136-4E16-4FDC-A22A-E5F46812F4CA CF8DDF98
InstallProtocolInterface: 6302D008-7F9B-4F30-87AC-60C9FEF5DA4E CF5D9800
```

Control is passed in BDS code by calling `StartImage`.

For some reason I couldn't print my logs to `debug by printk`. I verified that
I'm in correct code by placing assert, code was interrupted in correct place
but not serial log.

Trying to change `DebugLib` and provide correct `SerialIoLib` leads to reboot.
