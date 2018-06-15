---
layout: post
title: "Using Joda-Time in unit tests"
excerpt: Joda-Time is a Java library which provides a quality replacement for the Java date and time classes.
modified: 2015-02-15 08:01:33 +0300
categories: articles
tags: [java, joda-time, jdk, testing, unittesting]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
---

Prior to Java 8, the date and time classes included in the standard library are considered by many flawed. They're difficult for developers to use. For example, `java.util.Date` and `java.util.Calendar` are mutable, which is inconvenient in multi-threaded environments. I think [this](https://stackoverflow.com/questions/1969442/whats-wrong-with-java-date-time-api "StackOverflow question") StackOverflow question has some great answers explaining the situation.

[Joda-Time](http://www.joda.org/joda-time/ "Joda-Time project page") is a Java library which provides a quality replacement for the Java date and time classes. From Java SE 8 onwards, users are asked to migrate to `java.time` (JSR-310).

Using `Date` and `Calendar` classes in unit tests has one drawback. It is not possible to set the system time. Luckily Joda-Time provides an API to do that.

{% highlight java %}
DateTimeUtils.setCurrentMillisFixed()
{% endhighlight %}

If you decide to use that, be sure to reset the real system time in a `tearDown()` method. Otherwise there's a chance you have some other tests which expect the system time be the actual current time and depending on the test execution order you might see unexpected results.
