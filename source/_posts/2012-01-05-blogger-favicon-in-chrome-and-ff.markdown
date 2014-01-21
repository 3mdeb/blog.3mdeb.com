---
layout: post
title: "Blogger favicon in Chrome and FF"
date: 2012-01-05T00:22:00+01:00
comments: true
categories: blogger
---

I didn't imagine that placing the code for favicon can be so time consuming. Here are some tpis and tricks that can be useful to others.  
1. First find or create favicon ( [google](http://www.google.pl/search?q=favicon&um=1&ie=UTF-8&hl=en&tbm=isch&source=og&sa=N&tab=wi&ei=udsET4OiG8vO-Qbd27j5Cw&biw=1600&bih=732&sei=u9sET_miJsztOf-2uLYO#um=1&hl=en&tbs=isz:ex%2Ciszw%3A32%2Ciszh%3A32&tbm=isch&sa=1&q=favicon&pbx=1&oq=favicon&aq=f&aqi=g10&aql=&gs_sm=s&gs_upl=1959l2847l0l4598l6l5l0l0l0l3l407l1119l0.3.1.0.1l5l0&bav=on.2,or.r_gc.r_pw.,cf.osb&fp=7c76d7186cf3fdc4&biw=1600&bih=732) or [favicon.cc](http://favicon.cc/))  
2. Make sure that you have not set any favicon (Design-> Page Elelments-> Favicon)  
3. Add this code below </head> tag  
    

<link href='http://bsaves.com/HowHero/Blogger%20Templates/moonspot/images/favicon.ico' rel='shortcut icon' type='image/vnd.microsoft.icon' /> <link href='http://bsaves.com/HowHero/Blogger%20Templates/moonspot/images/favicon.ico' rel='icon'/> This tip comes from [here](http://widgetsforfree.blogspot.com/2009/08/how-to-change-favicon-in-blogger.html). It doesn't work on IE but this did not bother me.  
    
If you want put the favicon in the network to imageshack.us to receive a direct link from them you must register. For those who do not want to do that I propose to use available in Chrome the "inspect element" feature with its help you can easily read the entire value of the field "Direct Link" on imageshack.
