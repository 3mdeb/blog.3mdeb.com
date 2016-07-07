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

One flaw that this setup has is for project that last more then year. I don't
think making it `<customer>/<year>` improve things, because then I would have
tens of even hundred of directories in `projects`. Splitting it by year makes
searching focused. For now, when I deal with project longer then year I just
copy relevant part from previous year. By relevant part I mean something that I
really have to use, not one time reference. This can be for example SD card
image with one of releases.

Customer part is trivial, although sometimes can cause confusion. There are
situation where I start research not knowing what company I work for.
