---
layout: post
title: "Octopress depoying problem after migrating to *.github.io"
date: 2013-05-16 22:20
comments: true
categories: 
published: true
keywords: github pages, domain, github
description: Problem with deploying to github pages
---

My story was that I migrate to new environment and want to recover my octopress 
setup. I thought that it will be as simple as cloning my blog github repository
and start to work.

First problem that I experienced was lack of octopress repository as a remote 
branch - it is required to update your ocotpress blog to latest code. Fix was 
simple:
```
git remote add octopress git://github.com/imathis/octopress.git
```

After updating my blog I tried to deploy new article and I get:
```
## Pushing generated _deploy website
ERROR: Repository not found.
fatal: The remote end hung up unexpectedly
```

I figured out that my github pages were incorrectly set, so I run `rake 
setup_github_pages`. If you're user of github pages long enough you know that 
few weeks ago github decide to migrate domain from `*.github.com` to 
`*.github.io`. So I followed instructions in setup process and used `io` domain,
this is incorrect. If you have github repository for your pages with `com` in 
name you should use your orginal repository address. Repository was not changed.
This is obvious but I lost few minutes to figure out whats going on.
