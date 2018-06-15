---
layout: post
title: "Remmina: cannot connect to Windows server over RDP"
excerpt: What to do when suddenly you're unable to connecto to a Windows server with Remmina over RDP
modified: 2014-07-14 08:57:43 +0300
categories: articles
tags: [rdp, remmina, freerdp, remote desktop, linux, windows, ubuntu]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: true
---

It had been working yesterday and nothing to my knowledge had been changed. That's what I thought one day when I was unable to connect to a Windows server over RDP. I was using Remmina on Ubuntu. The remote server was up and running and passwords had not been changed. Doing a test from a Windows machine revealed that the issue must be specific to me because with the default Windows remote desktop client I was able to connect successfully. After a bit of googling I found a solution.

{% highlight bash %}
$ rm ~/.freerdp/known_hosts
{% endhighlight %}

I had to delete the `known_hosts` file. After that everything worked fine like before. I'm not exactly sure why this fixed it but at that moment my main concern was to get it working as soon as possible. 
