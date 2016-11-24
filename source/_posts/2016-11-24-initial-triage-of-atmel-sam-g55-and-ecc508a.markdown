---
author: Piotr Król
layout: post
title: "Initial triage of Atmel SAM G55 and Microchip ECC508A"
date: 2016-11-24 15:37:26 +0100
comments: true
categories: atmel samg55 ecc508a embedded firmware security aws iot
---

Some time ago (around August 2016) embedded community media were hit with hype
around simplified flow for AWS IoT provisioning
([1](http://www.embedded.com/electronics-products/electronic-product-reviews/safety-and-security/4442551/Crypto-chip-simplifies-AWS-IoT-security),
[2](http://www.embedded.com/electronics-blogs/max-unleashed-and-unfettered/4442574/Single-chip-end-to-end-security-for-IoT-devices-connected-to-Amazon-cloud),
[3](http://www.embedded.com/electronics-news/4423245/Microchip-goes-to-the-Cloud-for-IoT-design)).
I'm personally very interested in all categories related to those news:

* IoT - is 3mdeb business core and despite this term was largely abused these
  days, we just love to build connected embedded devices. Building this kind of
  devices is inherently related with firmware deployment, provisioning and
  update problems.

* AWS - truly it is had to find similar level of quality and feature-richness
  and because I was lucky to invest my time and work with grandfather of AWS
  IoT (namely 2lemetry ThingFabric) I naturally try to follow this trend and
  make sure 3mdeb customers use best in class product in IoT cloud segment. To
  provide that service we try to be on track with all news related to AWS IoT.

* Security - there will be not so much work for Embedded System Constultants if
  IoT will be rejected because of security issues. I'm sure I don't have to
  convince anyone about important of security. Key is to see typical flow that
  we face in technology (especially in security area): 

  ```
  mathematics -> 
  proof of concept software -> 
  mature software -> 
  hardware acceleration -> 
  hardware implementation
  ```

  AWS IoT cryptography is not trivial and doing it right is even more complex.
  Using crypt chips like ECC580A simplify whole workflow.

Initial idea for this blog post was to triage ECC508A with some Linux or mbed
OS enabled platform. Atmel SAM G55 seem to have support in mbed OS
[here](https://github.com/ARMmbed/target-atmel-samg55j19-gcc), but diving into
[CryptAuthentication](http://www.atmel.com/products/security-ics/cryptoauthentication/default.aspx)
with development stack that I'm not sure work fine is not best choice. That's
why I have to try stuff on Windows 10 VM and then after understanding things
better I will move to something more convenient.

I will mostly rely on [ATECC508A Node Authentication Example Using Asymmetric PKI](http://www.atmel.com/applications/iot/aws-zero-touch-secure-provisioning-platform/default.aspx?tab=documents)
Application Note.

What we need to start is:

* [Atmel Studio](http://www.atmel.com/tools/atmelstudio.aspx#download)
* [AT88CKECC-AWS-XSTK](http://www.atmel.com/tools/at88ckecc-aws-xstk.aspx)

## Atmel Studio

Welcome in the world of M$ Windows. I wonder who get idea of excluding Mac and
Linux users from Atmel SAM developers community, but this decision was really
wrong. Of course there are options like
[ASF](http://www.atmel.com/tools/AVRSOFTWAREFRAMEWORK.aspx) but this requires
much more work for setup and is probably not feasible for initial triage post.
Unfortunately number of examples in ASF is limited and I can't find anything
related to crypt or i2c.

Atmel Studio is obviously inspired or even build on Visual Studio engine.

### CryptoAuthentication Node Basic Example Solution

To make things simple `CryptoAuthentication Node Basic Example Solution.zip`,
which you can be downloaded
[here](http://www.atmel.com/applications/iot/aws-zero-touch-secure-provisioning-platform/default.aspx?tab=documents)
is 15MB and contain almost 2k of files. Download and unpack archive.

After starting Atmel Studio choose `Open Project...`, navigate to
CryptoAuthentication example and choose `node-auth-basic` you should get funny
pop-up that tells you to watch out for malicious Atmel Studio projects:

TODO: screenoshot

Then you have window with info `Please select your project`, so choose
`node-auth-basic`, then try `Build -> Rebuild Solution`, of course this doesn't
work out of the box.

One of problems that I faced was described
[here](http://asf.atmel.com/bugzilla/show_bug.cgi?id=3715) this is just
incorrect `OPTIMIZE_HIGH` macro. After fixing that both examples compile fine.

I realized that Atmel Studio use older ASF (3.28.1) then what is available
(3.32.0), but upgrading ASF leads to upgrading whole project and take time.
After upgrade you get report if everything went fine for your 2k files.

The problem with `node-auth-basic` is that it is not prepared for SAM G55.
Whole code in `AT88CKECC-AWS-XSTK` documents target SAM D21. So you have to
change target device and this is possible only after update. To change device
enter `node-auth-basic` project properties and got to `Device` tab, then use
`Change Device` find `SAMG` family and use `SAMG55J19`. Result should look like
this:

TODO: screenshot

Now we get more compilation errors:

```
Error       sam/sleepmgr.h: No such file or directory   node-auth-basic \
C:\(...)\cryptoauth-node-auth-basic\node-auth-basic\src\ASF\common\services\sleepmgr\sleepmgr.h 53
```

### CryptoAuthLib: Driver Support for Atmel CryptoAuthentication Devices

So I tried to download `CryptoAuthLib SAMD21 Test Host Project.zip` from the
same page - another project with > 1.5k files in it. 
