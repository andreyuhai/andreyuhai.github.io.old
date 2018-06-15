---
layout: post
title: "SEVERE: Error listenerStart - debugging Tomcat applications"
excerpt: "How to debug the dreaded 'Error listenerStart' error message"
modified: 2014-02-14 21:19:18 +0300
categories: articles
tags: [tomcat, java, error, logging, apache, catalina]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

When starting up Tomcat I'm sure at one point in time you've encountered a *'SEVERE: Error listenerStart'* message. This is caused by an application you have deployed. But what's the exact cause of this error?

Create `logging.properties`to `WEB-INF/classes/` with the following content:

{% highlight java %}
org.apache.catalina.core.ContainerBase.[Catalina].level = INFO
org.apache.catalina.core.ContainerBase.[Catalina].handlers = java.util.logging.ConsoleHandler
{% endhighlight %}

Now restart Tomcat and you should see a stacktrace about what went wrong in `catalina.out`.
