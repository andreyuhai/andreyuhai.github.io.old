---
layout: post
title: "Setting up a reverse SSH tunnel with a Raspberry Pi part 1"
excerpt: You need to access your work computer or network from your home. Maybe a Raspberry Pi can help you.
modified: 2015-09-17 20:09:28 +0300
categories: articles
tags: [ssh, tunneling, raspberry pi, vpn, linux, rdp, windows]
image:
  feature: 2015-09-18-setting-up-a-reverse-ssh-tunnel-with-a-raspberry-pi/pi_cover.jpg
  credit: http://www.kahvel.ee/
  creditlink: http://kahvel.ee/wp-content/uploads/2013/07/5456b7bac8c8b4f666ba66341e321ec1.jpg
comments: true
share: true
published: true
aging: true
---

Think of a following scenario. You need to access your work computer from your home. You have a small office with no dedicated IT support. Probably you don't have a VPN which you could use to connect to your work network. But you have a spare Raspberry Pi lying around. You could put that into use and create a connection to your work machine by creating a reverse SSH tunnel.

**What you're going to need:**

*  a working Raspberry Pi
*  physical access to your work network
*  a server outside of your work network
*  basic understanding of SSH

## Setting up an outside server

You're going to need an outside server for your Pi to connect into. If you already have a box with a static IP and SSH running then you're good to go. If not, then you could create a cheap VPS at [DigitalOcean](https://www.digitalocean.com/?refcode=0b1905c10185 "link to DigitalOcean") for example. Additionally, read [this post]({{site.url}}/articles/5-steps-to-improve-ssh-security/ "5 steps to improve SSH security") about further securing your SSH server.

## Setting up the Pi

To begin with, plug in your Pi and connect it to your office network. Then you should check if you can connect to your server via SSH. If you have not yet set up key-based authentication then this should be the next step. It is needed so the Pi can ssh into your server without being prompted for a password. [Ubuntu help pages](https://help.ubuntu.com/community/SSH/OpenSSH/Keys "ubuntu help page for openssh") have a step-by-step guide on how to do that.

## Creating a reverse SSH tunnel

Now let's create a tunnel from your Pi to the Linux server.

{% highlight bash %}
$ ssh -N -R 2222:localhost:22 user@yourserverhostname
{% endhighlight %}

Pi is connecting to your server using the username `user`. The `-R` flag specifies that the given port on the remote (server) host is to be forwarded to the given host and port on the local side. Meaning that all connections made to port `2222` on your server are forwarded to the Pi on port `22`. The `-N` flag tells SSH to not execute a remote command. It is useful for port forwarding. For further reading about SSH tunnels, check out [this StackExchange answer](http://unix.stackexchange.com/a/46271/134898 "How does reverse SSH tunneling work?").

## In the next part

So far we've set up the Pi which connects out to an external server. From there it is possible to access the Pi from wherever you have access to the external server. That means you have access to your work network. In the [next part]({{site.url}}/articles/setting-up-a-reverse-ssh-tunnel-with-a-raspberry-pi-part-2/ "link to part 2 of the article") let's look at an example use case on how to connect to a Windows workstation via RDP. That requires some SSH tunneling.
