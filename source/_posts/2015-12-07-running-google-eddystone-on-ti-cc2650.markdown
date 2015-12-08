---
layout: post
title: "Running Google Eddystone on TI CC2650"
date: 2015-12-07 22:43:31 +0100
comments: true
categories: cc2650 ti bluetooth
---

According to documentation on GitHub, Eddystone should be ready to use on [TI CC2640](https://github.com/google/eddystone/tree/master/eddystone-url/implementations/TI-CC2640),
which is compatible with CC2650. Instruction is straight forward but whole
environment cannot be built without solving couple issues.

From time to time I'm using TI Code Composer Studio to develop software for
CC3200. Not sure if this is related but my Windows 10 refused to install BLE
Stack 2.1 or older version.

I started thread to solve that problem [here](https://e2e.ti.com/support/wireless_connectivity/f/538/t/475341)

Meanwhile I decided to setup Eddystone and CC2650 development environment under
Linux. Big kudos for previous work to Norman Mackenzie for [this post](https://e2e.ti.com/support/wireless_connectivity/f/538/p/412962/1495231#1495231).

## Code Composer under Linux

First download CCS from
[here](http://processors.wiki.ti.com/index.php/Download_CCS). Unzip and start
installer. Most probably you will get error message like this:




