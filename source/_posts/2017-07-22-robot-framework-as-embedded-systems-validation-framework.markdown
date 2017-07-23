---
author: Piotr Król
layout: post
title: "Robot Framework as embedded systems validation framework"
date: 2017-07-22 22:42:44 +0200
comments: true
categories: apu2 python robotframework validation automation
---

Recently we started to prepare to [ECC2017](https://ecc2017.coreboot.org/)
conference one of topics that we considered was system for development and
validation automation.

As maintainers of PC Engines platforms in coreboot we fix quite a lot of bugs,
but to take full responsibility for out code everything should be validate each
time we do release. Limited resources leads us to automation and as Python
enthusiast we decided to evaluate Robot Framework as first candidate that came
to mind.

When preparing to mentioned conference I found that we lack PXE server from
which I could run or install needed OS. Also there was no configuration when I
could use diskless boot to try recent Linux. I started to fight with PXE server
configuration, but then I realized that without DHCP I have to provide booting
information every time. This was very annoying and I know I can do better from
my early days as validation engineer in Intel. That's why I decided fight it in
a more structured way.

## Robot Framework first try

Project by itself seems to be very popular and at first glance have good
design. I decided to start with virtualenv:

```
[23:00:11] pietrushnic:storage $ virtualenv robot-venv
Running virtualenv with interpreter /usr/bin/python2
New python executable in /home/pietrushnic/storage/robot-venv/bin/python2
Also creating executable in /home/pietrushnic/storage/robot-venv/bin/python
Installing setuptools, pkg_resources, pip, wheel...done.
[23:00:29] pietrushnic:storage $ source robot-venv/bin/activate
(robot-venv) [23:00:36] pietrushnic:storage $ pip install robotframework
Collecting robotframework
  Downloading robotframework-3.0.2.tar.gz (440kB)
    100% |████████████████████████████████| 450kB 2.2MB/s 
Building wheels for collected packages: robotframework
  Running setup.py bdist_wheel for robotframework ... done
  Stored in directory: /home/pietrushnic/.cache/pip/wheels/b4/9b/b2/75d7e5f88f21673eed3472266a1f4e72672328cc174655a1b6
Successfully built robotframework
Installing collected packages: robotframework
Successfully installed robotframework-3.0.2
```

Verification:

```
(robot-venv) [23:00:46] pietrushnic:storage $ robot --version
Robot Framework 3.0.2 (Python 2.7.13 on linux2)
```

## Quick start guide

```
sudo apt-get install docutils
git clone https://github.com/robotframework/QuickStartGuide.git
cd QuickStartGuide
robot QuickStartGuide.rst
```

Output should look like this:

```
==============================================================================
QuickStart
==============================================================================
User can create an account and log in                                 | PASS |
------------------------------------------------------------------------------
User cannot log in with bad password                                  | PASS |
------------------------------------------------------------------------------
User can change password                                              | PASS |
------------------------------------------------------------------------------
Invalid password                                                      | PASS |
------------------------------------------------------------------------------
User status is stored in database                                     | PASS |
------------------------------------------------------------------------------
QuickStart                                                            | PASS |
5 critical tests, 5 passed, 0 failed
5 tests total, 5 passed, 0 failed
==============================================================================
Output:  /home/pietrushnic/storage/wdc/projects/2017/pcengines/apu/src/QuickStartGuide/output.xml
Log:     /home/pietrushnic/storage/wdc/projects/2017/pcengines/apu/src/QuickStartGuide/log.html
Report:  /home/pietrushnic/storage/wdc/projects/2017/pcengines/apu/src/QuickStartGuide/report.html
```

What is great about that. It is clean and can be easily understood. In addition
it generate log and report. Report and log files are clean and eye-catching. So
far so good.

## Let's try something real on PC Engines APU2

My typical output on minicom during boot was:

```
PC Engines apu2
coreboot build 07/18/2017
BIOS version
4080 MB ECC DRAM

SeaBIOS (version rel-1.10.2.1)
Press F10 key now for boot menu, N for PXE boot
```

After `N for PXE boot` show script should sent `N` or `n` and I should go to
`iPXE>` prompt.

Initially I thought about using [robotframework-seriallibrary](https://github.com/whosaysni/robotframework-seriallibrary),
but its limitation lead me to search for different solution and look like using
`ser2net` and `telnet` library seems to be great idea. This solution was suggested on [mailing list](https://groups.google.com/d/msg/robotframework-users/r0xvLtGNgno/TI0suLOlNL4J).

```
sudo apt-get install ser2net
```

Quick test with config file `ser2net_apu.cfg`

```
13542:telnet:600:/dev/ttyUSB0:115200 8DATABITS NONE 1STOPBIT
```

```
[0:19:39] pietrushnic:pcengines $ telnet localhost 13542
Trying ::1...
Connected to localhost.
Escape character is '^]'.
PC Engines apu2
coreboot build 07/18/2017
BIOS version
4080 MB ECC DRAM


telnet> q
Connection closed.
```

### Telnet module for Robot Framework

After playing some time I get 


