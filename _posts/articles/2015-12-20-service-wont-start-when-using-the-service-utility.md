---
layout: post
title: "Service won't start when using the service utility"
excerpt: I debugged a service which didn't start when using the service utility but was working fine when calling the init script directly.
modified: 2015-12-20 12:30:56 +0200
categories: articles
tags: [upstart, system v, service, linux, ubuntu, mule, esb, init]
image:
  feature: 2015-12-16-service-wont-start-when-using-the-service-utility/cover.jpg
  credit: Pronunciation Academy
  creditlink: https://www.youtube.com/watch?v=PX8Ji60zwf4
comments: true
share: true
published: true
aging: true
---

Recently I learned something new and interesting regarding the Service utility. I was trying to run Mule on Ubuntu 14.04.

>Mule ESB is a lightweight Java-based enterprise service bus (ESB) and integration platform that allows developers to connect applications together quickly and easily, enabling them to exchange data.

To get the latest version I downloaded the binaries from Mule's website. Since I wanted it to be run as a service, I created the following init script (in `/etc/init.d/mule`).

{% highlight bash %}
#!/bin/bash

JAVA_HOME=/usr/lib/jvm/java-8-oracle
PATH=$PATH:$JAVA_HOME/bin

# Set Mule related environment
MULE_HOME=/opt/mule
PATH=$PATH:$MULE_HOME/bin
RUN_AS_USER=mule

# Export environment variables
export JAVA_HOME MULE_HOME RUN_AS_USER PATH

# Invoke Mule
$MULE_HOME/bin/mule $1
{% endhighlight %}

It sets some environment variables which are needed by Mule to run. Most notably it initializes `RUN_AS_USER` to the user I want Mule to run as.

When I call `service mule start` I see nothing happening. Nothing in the logs and no sign when using the `ps` command. But when calling `/etc/init.d/mule start`, Mule starts running fine.

**Why is it that when calling the init script directly, Mule starts running fine but when using the service utility, nothing happens?** Looking at the [startup script](https://github.com/mulesoft/mule/blob/91f36d45a0b0744cd36f2c3037618aa7f5a655b1/distributions/standalone/src/main/resources/bin/mule "Mule startup script") for Mule, on line 419 the script is relaunched if it needs to be run as another user.

{% highlight bash %}
RELAUNCH_CMD="$REALPATH $@"
su -m $RUN_AS_USER -c "$RELAUNCH_CMD"
{% endhighlight %}

The man page for `service` says that it removes most environment variables. So my initial guess was that calling `su` depends on the environment somehow.

An excerpt from `service` man page:

>service runs a System V init script or upstart job in as predictable an environment as possible, removing most environment variables and with the current working directory set to /.

After some time spent on debugging I discovered that the user `mule` did not have a shell. It was set to `/bin/false`. The `-m` flag for `su` preserves the environment, so when I called `/etc/init.d/mule start` the `SHELL` environment variable was set from my current session and passed down to `su`.

>The `SHELL` variable describes the shell that will be interpreting any commands you type in. In most cases, this will be bash by default, but other values can be set if you prefer other options.

Conversely, when calling `service mule start` the environment is cleaned. Therefore the environment did not contain a `SHELL` variable and the user `mule` did not have a shell as well. So it seems no command was executed.

This issue got resolved when I added the following lines to the init script.

{% highlight bash %}
# Set shell
SHELL=/bin/bash
{% endhighlight %}

Now there is a shell which can interpret commands and Mule starts up fine when using the `service` utility.
