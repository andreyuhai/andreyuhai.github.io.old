---
layout: post
title: "Sharing a local folder during a remote desktop session with RDesktop"
excerpt: "Rdesktop is an open source client for Windows Remote Desktop Services, capable of natively speaking Remote Desktop Protocol (RDP) in order to present the user's Windows desktop"
modified: 2014-06-02 07:54:24 +0300
categories: articles
tags: [rdp, remote desktop, rdesktop, linux, windows, share, remmina]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: true 
---

[Rdesktop](http://www.rdesktop.org/ "rdesktop homepage") is an open source client for Windows Remote Desktop Services, capable of natively speaking Remote Desktop Protocol (RDP) in order to present the user's Windows desktop. Not long ago I needed an easy way to copy files from the remote machine to my local machine and vice versa. The default RDP application on Windows is capable of doing that with just `ctrl+c` and `ctrl+v`. Unfortunately Linux RDP clients do not support such feature (at least I'm not aware of it). The closest solution I found was to share a local folder to the remote machine.

{% highlight bash %}
$ rdesktop -r disk:tmp=/home/user/Desktop <remote ip address>
{% endhighlight %}

For example, the previous command creates a new RDP session and shares my desktop to the remote Windows machine. The same solution can be achieved with [Remmina](http://remmina.org/ "Remmina homepage") as well. 