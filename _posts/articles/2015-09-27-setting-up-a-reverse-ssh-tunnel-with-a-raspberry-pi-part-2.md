---
layout: post
title: "Setting up a reverse SSH tunnel with a Raspberry Pi part 2"
excerpt: In part 2 I promised to show an example use case on how to connect to a Windows Server over RDP via a reverse SSH tunnel.
modified: 2015-09-27 19:25:14 +0300
categories: articles
tags: [raspberry pi, ssh, linux, rdp, windows server, tunneling, networking]
image:
  feature: 2015-09-27-setting-up-a-reverse-ssh-tunnel-with-a-raspberry-pi-part-2/cover.jpg
  credit: http://wallpaperstock.net/
  creditlink: http://wallpaperstock.net/tunnel-in-zurich_wallpapers_27851_1280x800_1.html
comments: true
share: true
published: true
aging: true
---

Last time we got a Raspberry Pi set up and a VPS ready and accessible via SSH. If you have not read [part 1]({{site.url}}/articles/setting-up-a-reverse-ssh-tunnel-with-a-raspberry-pi/ "part 1 of setting up a reverse ssh tunnel with a raspberry pi") then I strongly encourage you to do that. In part 2 I promised to show an example use case on how to connect to a Windows Server over RDP via a reverse SSH tunnel.

## Testing SSH access

First, let's make sure that the reverse SSH tunnel is working as expected. I'm going to create an SSH session to the VPS and from there I'm going to create another SSH session to the Pi.

Connecting to the external server:
{% highlight bash %}
$ ssh user@serverhostname
{% endhighlight %}

From there let's create another SSH session to our Pi which is inside the office network:
{% highlight bash %}
$ ssh pi@localhost -p 2222
{% endhighlight %}

Why connect to `localhost` on port `2222`? Recall from [part 1]({{site.url}}/articles/setting-up-a-reverse-ssh-tunnel-with-a-raspberry-pi/ "part 1 of setting up a reverse ssh tunnel with a raspberry pi") that we created a reverse SSH tunnel from the Pi to the server. We defined that all connections from our server to `localhost` on port `2222` should be forwarded to the Pi.

If you were able to connect to your Pi then the tunnel is working.

## Tunneling RDP traffic

For the sake of the example I'm going to assume the IP of the Windows Server at the office network is `192.168.1.10` and the server outside of the network can be accessed via `myserver.com`. Two SSH tunnels are needed to forward traffic from the client machine to `myserver.com` and from there to the Pi.

### External server -> Raspberry Pi -> Windows Server

First of all, I'm going to create a tunnel to forward RDP traffic from `myserver.com` to the Windows Server.

{% highlight bash %}
$ ssh -L 3000:192.168.1.10:3389 pi@localhost -p 2222
{% endhighlight %}

This forwards all connections made to `localhost` on port `3000` to the Windows Server (`192.168.1.10`) using port `3389`. `-L` flag specifies that the given port on the local (client) host is to be forwarded to the given host and port on the remote side. It is achieved by creating an SSH tunnel to the Pi via a reverse SSH tunnel we set up previously. Note that `pi@localhost -p 2222` in the command is the same used previously to test the SSH tunnel.

### Client machine -> External server

From the client machine an SSH tunnel must be created to forward all connections made to `localhost` port `5000` (just a random port I chose) to the external server.

{% highlight bash %}
$ ssh -L 5000:localhost:3000 user@myserver.com
{% endhighlight %}

Now when you connect your RDP client to `localhost:5000`, this connection is forwarded to `myserver.com` and from there a new connection is made to `localhost:3000`. Looking at the preceding step, you can see that this will be in turn forwarded the Windows Server.

## Recap

In short, the following steps were done:

1. From the Pi create a reverse SSH tunnel to an outside server.
2. From an outside server tunnel incoming traffic to the Pi and from there to the Windows Server.
3. From the client machine tunnel RDP traffic to an outside server.

## Summary

There are a couple of more things you need to do to make it a reliable solution. Obviously the steps need to be automated. It can get cumbersome to create two SSH tunnels over and over again. Additionally the reverse SSH tunnel needs to be kept alive. Network errors and power outages can occur which will kill the tunnel. This means you have to physically go to the Pi and redo the tunnel. I would recommend creating a cronjob which checks if the tunnel is up and recreates it if it is not.
