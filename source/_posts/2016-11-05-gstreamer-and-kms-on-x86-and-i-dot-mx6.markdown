---
author: Piotr Król
layout: post
title: "GStreamer and KMS on x86 and i.MX6"
date: 2016-11-05 15:45:39 +0100
comments: true
categories: imx6 x86 embedded gstreamer
---

After [initial research](/2016/11/01/chromium-gstreamer-backed-for-i-dot-mx6-preparation/) I
decided to focus on GStreamer for i.MX6. Googling various resources I found
very interesting blog by [Víctor Jáquez](https://blogs.igalia.com/vjaquez/).

Since these days hardware video decoding is hot topic in embedded system
development and GStreamer seems to be key component that can utilize kernel
drivers that expose decoding capability I tried to reproduce one of Victor
experiments on x86 with VA-API. If this would be successful I will move on with
exercising similar approach but with CODA960 on i.MX6.

## GStreamer development environment

Obviously tried to use recent GStreamer code and since this is not small
project some order is needed. Victor pointed to very interesting project
[gst-uninstalled](https://arunraghavan.net/2014/07/quick-start-guide-to-gst-uninstalled-1-x/)
that is advertised as `virtualenv` for GStreamer. Nevertheless I won't trash my
Debian with random packets for another evaluation project that's why I created
[gst-uninstalled-docker]().

```

```
