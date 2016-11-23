---
author: Piotr Król
layout: post
title: "Starting with mdeb OS for Linux and command line enthusiast"
date: 2016-11-23 16:01:48 +0100
comments: true
categories: mbed embedded linux
---

When I first time read about mbed OS I was really sceptical, especially idea of
having web browser as my IDE and compiler in the cloud seems to be very scary
to me. ARM engineers proved to provide high quality products, but this was not
enough to me. Then I heard very good words about mbed OS IDE from Jack Ganssle,
this was still not enough. Finally customers started to ask about this RTOS and
I had to look deeper.

TODO: note about other RTOSes

Below I want to present Linux user experience from my first contact with mbed OS.

## First contact

I have to say that at first glance whole system is very well documented with
great look and feel. [Main site](https://www.mbed.com/en/) requires 2 clicks to
be in correct place for Embedded System Engineer. In general we have 3 main
path when we choose developer tools: Online IDE, mbed CLI and 3rd party. Last
covers blasting variety of IDEs including Makefile and Eclipse CDT based GCC
support.

Things that are annoying during first contact we web page:

* way to contribute documentation is not clear
* there is no description how to render documentation locally
* can't upload avatar on forum - no information what format and resolution is
  supported

But those are less interesting things. Going back to development environment
for me 2 options where interesting mbed CLI and plain Makefile.

### mbed CLI

I already have setup vitrualenv for Python 2.7:

```
pip install mbed-cli
```

First thing to like in `mbed-cli` is that it was implemented in Python. Of
course this is very subjective since I'm familiar with Python, but it good to
know that I can hack something that doesn't work for me. Is is [Open Source](https://github.com/ARMmbed/mbed-cli).

I also like the idea of mimicking git subcommands. More information about mbed
CLI can be found in
[documentation](https://docs.mbed.com/docs/mbed-os-handbook/en/5.2/dev_tools/cli/#using-mbed-cli).

It is also great that mbed CLI tries to manage whole program dependencies in
structured way, so no more hassle with external libraries versioning and trying
to keep sanity when you have to clone your development workspace. Of course
this have to be checked on battlefield, since documentation promise may be not
enough.

So first thing that hit me when trying to move forward was this message:

```
$ mbed new mbed-os-program                                                  
[mbed] Creating new program "mbed-os-program" (git)
[mbed] Adding library "mbed-os" from "https://github.com/ARMmbed/mbed-os" at branch latest
[mbed] Updating reference "mbed-os" -> "https://github.com/ARMmbed/mbed-os/#d5de476f74dd4de27012eb74ede078f6330dfc3f"
[mbed] Auto-installing missing Python modules...
[mbed] WARNING: Unable to auto-install required Python modules.
---
[mbed] WARNING: -----------------------------------------------------------------
[mbed] WARNING: The mbed OS tools in this program require the following Python modules: prettytable, intelhex, junit_xml, pyyaml, mbed_ls, mbed_host_tests, mbed_greentea, beautifulsoup4, fuzzywuzzy
[mbed] WARNING: You can install all missing modules by running "pip install -r requirements.txt" in "/home/pietrushnic/tmp/mbed-os-program/mbed-os"
[mbed] WARNING: On Posix systems (Linux, Mac, etc) you might have to switch to superuser account or use "sudo"
```

This appeared to be some problem with my distro:

```
(...)
    ext/_yaml.c:4:20: fatal error: Python.h: No such file or directory
     #include "Python.h"
                        ^
    compilation terminated.
(...)
```

This indicate lack of `python2.7-dev` package, so:

```
sudo apt-get install python2.7-dev
```
