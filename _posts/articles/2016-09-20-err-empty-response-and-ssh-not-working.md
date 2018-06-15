---
layout: post
title: "ERR_EMPTY_RESPONSE and SSH not working"
excerpt: Chrome is returning ERR_EMPTY_RESPONSE, Git and SSH connection hangs at expecting SSH2_MSG_KEX_ECDH_REPLY. Seems to be an MTU/fragmentation problem.
modified: 2016-09-20 16:17:38 +0200
categories: articles
tags: [ssh, chromium, chrome, git, mtu, ifconfig]
image:
  feature: 2016-09-19-err-empty-response-and-ssh-not-working/cover.png
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

I was browsing the Internet on a WiFi network I have never used before. Everything seemed fine until I found that some web apps did not work correctly. Initially I thought the web app had problems and didn't pay much attention to it, hoping it would be fixed soon. Then I realized that the same web apps work correctly on my phone. Something smells fishy and I started to debug.

Have I updated anything major recently? Not that I know of. Did I try a different browser? Yes, same issues. For some websites Chrome says the website didn't send any data (ERR_EMPTY_RESPONSE).

<figure>
	<a href="{{ site.url}}/images/2016-09-19-err-empty-response-and-ssh-not-working/error.png" class="image-popup"><img src="{{ site.url}}/images/2016-09-19-err-empty-response-and-ssh-not-working/error.png" alt="Error in Chromium"></a>
	<figcaption>Error screen in Chromium</figcaption>
</figure>

At first I thought ERR_EMPTY_RESPONSE can be caused by ad-block software. After disabling all browser plugins, the issue still persisted. Then I used a different WiFi network by creating a hotspot with my phone. Surprisingly the issues were resolved. Something wrong with the network then, right? Probably. Since I was not going to use the problematic WiFi network for long, I decided that I can manage without some web apps for now.

All went well until I started to use Git. As luck would have it, pushing commits did not work. Turns out, SSH does not make the connection. Running SSH with verbose mode I saw that it got stuck at expecting SSH2_MSG_KEX_ECDH_REPLY.

After some further *googling* I found that there might be a [MTU/fragmentation problem](http://www.snailbook.com/faq/mtu-mismatch.auto.html).

> At each routing hop, IP packets bigger than the outgoing interface's Maximum Transmission Unit (MTU) get fragmented. Only the first fragment has TCP port numbers. Firewalls often behave badly in the presence of packet fragmentation, dropping everything but the first fragment since the subsequent ones can't be matched against the firewall rules. Some NAT configuration (eg many-to-one NAT or port address translation) can't match the fragments against their translation state tables.

Setting my network interface's MTU to 576 solved all issues. SSH started working correctly and no issues with Chrome either.

{% highlight bash %}
# view ifconfig to see all interfaces
# eth0 is the interface I was using
$ ifconfig eth0 mtu 576
{% endhighlight %}
