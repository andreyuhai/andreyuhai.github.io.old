---
layout: post
title: "Solving MySql JDBC CommunicationsException"
excerpt: "I faced a com.mysql.jdbc.exceptions.jdbc4.CommunicationsException and this is how I solved it."
modified: 2013-12-17 17:23:11 +0300
categories: articles
tags: [java, mysql, jdbc, exception, dbcp, spring]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

I was trying to connect to a MySql database but every once in a while I got the following exception

{% highlight java %}
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
{% endhighlight %}

As usual, I opened a new browser tab and started googling for possible solutions. To my amazement, I found a lot of proposed solutions. I'm going to list 2 of them that worked for me.

## 1. Add *?autoReconnect=true* to you JDBC URL

{% highlight java %}
jdbc:mysql://localhost:3306/?autoReconnect=true
{% endhighlight %}

## 2. Using DBCP as connection pool with Spring

Add the following entries to your datasource definition (the last 2 property elements):

{% highlight xml %}
<bean id=”datasource” destroy-method=”close”>
	<property name=”driverClassName” value=”${.jdbc.driver}” />
	<property name=”url” value=”${jdbc.url}” />
	<property name=”username” value=”${jdbc.username}” />
	<property name=”password” value=”${jdbc.password}” />
	<property name=”validationQuery” value=”SELECT 1″ />
	<property name=”testOnBorrow” value=”true” />
</bean>
{% endhighlight %}
