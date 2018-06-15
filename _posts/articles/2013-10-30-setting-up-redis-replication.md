---
layout: post
title: "Setting up Redis replication"
excerpt: "The goal of this post is to install Redis on both of them and set up a master-slave replication configuration"
modified: 2013-10-30 20:41:18 +0300
categories: articles
tags: [redis, replication, ubuntu, master, slave, redis-cli, redis-server]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

I created 2 ubuntu server virtual machines. The goal of this post is to install Redis on both of them and set up a master-slave replication configuration. [Redis](http://redis.io/ "Redis homepage") is an open source, BSD licensed, advanced key-value cache and store.

To start, follow these steps:

* Install redis-server on both VMs
* To test if current setup is working, run `redis-cli`
* Then run ´SET keyname "fido"´ this will set a new key and its value
* To retrieve a keu value, run `GET keyname`, and the result should be "fido"  

## Configuring Redis

Now that we know Redis is working, let's continue configuring Redis. Configuration file is located at `/etc/redis/redis.conf`.

To make `redis-server` connectable from the outside you need to open `redis.conf` and find the place where a bind  declaration is set. Remove or comment it out. If there is no bind statements, then all network interfaces will be listened for incoming connections. Save the file, close it and restart redis-server.

{% highlight bash %}
$ service redis-server restart
{% endhighlight %}

## Making an instance of Redis a slave

Open the configuration file for Redis slave instance and add the following line

<pre>
slaveof 192.168.10.101 6379
</pre>

Default configuration should already contain a section for slaves which is commented out. Change the IP and port number to Redis master server IP and port and then restart the slave instance.

{% highlight bash %}
$ service redis-server restart
{% endhighlight %}

## Redis logs

To see if the configuration is working, check Redis logs. They can be found at `/var/log/redis/`. If everything succeeded, then you shoud see something like the following in your logs:

<pre>
MASTER <-> SLAVE sync started: SYNC sent...
</pre>

If you see "unable to connect to master", then make sure you can ping the master server.

{% highlight bash %}
$ redis-cli -h <master-host-or-ip> ping
{% endhighlight %}

The result should be: PONG. If ping is unsuccessful, then check if master Redis server is running and it is listening on the correct network interface.

## Testing

On the master server, set a new value. On the slave server, get the same value with the same key.

Master:
{% highlight bash %}
$ SET myValue "foo"
{% endhighlight %}

Slave:
{% highlight bash %}
$ GET myValue
{% endhighlight %}

And you should see "foo" as the result.  
