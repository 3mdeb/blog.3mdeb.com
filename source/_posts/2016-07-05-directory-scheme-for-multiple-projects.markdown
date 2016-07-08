---
layout: post
title: "Directory scheme for multiple projects"
date: 2016-07-05 23:31:33 +0200
comments: true
categories: productivity
---

How to keep clean organization while working on multiple projects ?
-------------------------------------------------------------------

Answer to this question is dependant on workflow and nature of projects itself.

Below I would like to present my approach to managing sanity while having
multiple projects going simultaneously. This would be Embedded Systems
Consultant view, but I think it can be adapted to other programmers workflow.

Directory organization
----------------------

Usually I have up to 10 projects from external customer running and ~3
internal. Obviously better organization minimize overhead related with
searching and wondering where to put recently obtained file.

My directory structure looks like that:

```
${HOME}/projects/<year>/<customer>/<project-name>
```


## Customer/year order

One flaw that this setup has is for project that last more then year. I don't
think making it `<customer>/<year>` improve things, because then I would have
tens of even hundred of directories in `projects`. Splitting it by year makes
searching focused. For now, when I deal with project longer then year I just
copy relevant part from previous year. By relevant part I mean something that I
really have to use, not one time reference. This can be for example SD card
image with one of releases.

## Customer

Customer part is trivial, although sometimes can cause confusion. There are
situation where I start research not knowing what company I work for, because I
was reached not from company domain. There are also cases when someone reach me
over freelance portals (Upwork, Guru etc.) that information provided are
outdated or simply invalid.

Having correct customer name is important only at invoicing stage, before that
if I'm not clear I just place some made up string that can uniquely identify
customer. Usually this is company name and contact person name, if company
unknown.

## Project name

Usually prototype projects doesn't have marketing name, but project can be
called by SoC/CPU/dev board + main feature ie. a20_camera, bbb_canbus_reader
etc.

What most embedded projects needs ?
-----------------------------------

After couple years I found that couple thing are typically needed:

* `logs` - this directory is used most of the times, I tend to run `minicom` in
  it with enabled logging, you never know when you will need information form
  this directory, naming convention for log files is something I still struggle

* `images` - this is directory for OS images, typically I have here SD card
  images and ISO images of distros used in project, sometimes you may end up
  keeping multiple instance of the same OS in various projects, but with 1TB
  disc this should not be big concern, you can always search for duplicates,
  knowing where you OS is and avoiding downloading it again can save some time

* `releases` - 


