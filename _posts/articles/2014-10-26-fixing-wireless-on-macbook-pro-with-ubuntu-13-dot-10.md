---
layout: post
title: "Fixing wireless on Macbook Pro with Ubuntu 13.10"
excerpt: "Downgrading Broadcom's proprietary driver and making sure no updates are applied to it."
modified: 2014-10-26 16:57:53 +0300
categories: articles
tags: [ubuntu, raring, saucy, apple, macbook, wireless, wifi, broadcom, bcmwl-kernel-source]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

Surprisingly, installing Ubuntu 13.04 on a MacbookPro10,2, almost everything seemed to work out of the box. But after upgrading to 13.10, there was a one major flaw which is *very* annoying if not fixed. Wifi connection seemed to be dropping a lot of packets. For me it was at most 50% packet loss. The issue seems to be with Broadcom’s proprietary drivers. Apparently the previous version of Ubuntu shipped with an older version of the driver which seemed to work fine.

The solution is to downgrade the Broadcom driver. I followed instructions posted in [this blog post](https://nick.groenen.me/posts/2013/11/09/fixing-the-broadcom-bcm4331-wireless-drivers-on-ubuntu-1310/ "Nick Groenen's blog"). It covers how to successfully downgrade the driver and make sure no future update is applied to it.

There's also a bug reported on [Launchpad](https://bugs.launchpad.net/ubuntu/+source/bcmwl/+bug/1289857 "link to bug report on launchpad").
