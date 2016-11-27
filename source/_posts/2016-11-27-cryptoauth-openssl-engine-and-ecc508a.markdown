---
author: Piotr Król
layout: post
title: "CryptoAuth OpenSSL engine and ECC508A"
date: 2016-11-27 23:20:31 +0100
comments: true
categories: openssl ecc508a security iot embedded linux
---

After [failure of my previous approach](2016/11/24/initial-triage-of-atmel-sam-g55-and-ecc508a/) I decide to
switch to something that seems to be more feasible. As project objectives states:

> Customers may buy un-programmed ATECC508A devices, download this project, build it, and establish a TLS1.2 connection without writing any code.
> Customers that buy personalized devices should be able to use these devices without writing any code.

Of course repository is getting older and no update was provided since February 2016. 

There were just 4 contributors where Atmel employee plus one consultant seemed
to provide most of the work. One closed issue, no merge requests no valuable
forks and 13 stars. I would not say this is vibrant community.

My goal was to confirm if project objectives are true and implement AWS Zero
Touch provisioning for Embedded Linux system. Method which utilize ECC508A was
overhyped by embedded news media in August 2016 what I described in [previous
post](2016/11/24/initial-triage-of-atmel-sam-g55-and-ecc508a/).

## Compilation

```
git clone https://github.com/AtmelCSO/cryptoauth-openssl-engine.git
cd cryptoauth-openssl-engine
make
```

## Cryptoauth Xplained Pro

Luckily Atmel website provide enough information to continue evaluation of
ECC508A.


