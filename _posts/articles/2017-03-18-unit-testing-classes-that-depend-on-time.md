---
layout: post
title: "Unit testing classes that depend on time"
excerpt: Dependency inversion principle states that we should depend upon abstractions. Taking that into account, time should also be considered as a dependency.
modified: 2017-03-18 17:01:27 +0200
categories: articles
tags: [java, java 8, time, clock, testing, dependency injection]
image:
  feature: 2017-02-26-unit-testing-classes-that-depend-on-time/cover.jpg
  credit: Wil Stewart
  creditlink: https://unsplash.com/@wilstewart3?photo=tB4-ftQ4zyI
comments: true
share: true
published: true
aging: true
---

Sometimes in unit testing you need to have control of the current time. But when your code that's under test depends on the current system time, it's not always that easy to achieve. As developers, we're taught to favor loosely coupled code. I'm sure you've heard of the [dependency inversion principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle). It states that you shouldn't depend upon concrete classes but you should depend upon abstractions. Taking that into account, [time should also be considered as a dependency](https://stackoverflow.com/questions/5622194/time-dependent-unit-tests/5622222#5622222 "Time dependent unit tests").

## Current time as a dependency

Think of the following situation. You're designing a class which needs to check the current time. Let's say you're working on a message board (e.g. [Discourse](https://www.discourse.org/)) and you would like to congratulate the user on his or her birthday when they log in. The class would need to check the current date and compare it against the user's entered birthday. If they're equal, it should display birthday congratulations.

But how would you unit test that class? If your implementation retrieves the current system time, then you have tightly coupled your class to the server's clock. To avoid that, think of the current time as one of many dependencies your class has. You can use familiar techniques such as [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection "Dependency Injection") to pass a replaceable clock to your class. In a production environment, your class can use a clock that retrieves the current system time. In unit tests, you can pass in a clock that reports a given fixed time.

## `java.time.Clock`

Java 8 introduced a [new date and time API](http://www.oracle.com/technetwork/articles/java/jf14-date-time-2125367.html "Java SE 8 Date and Time"). It includes a replaceable [Clock](https://docs.oracle.com/javase/8/docs/api/java/time/Clock.html) which is specifically designed to be used as a dependency.

> Instances of this class are used to find the current instant, which can be interpreted using the stored time-zone to find the current date and time. As such, a clock can be used instead of `System.currentTimeMillis()` and `TimeZone.getDefault()`.

Instead of retrieving the current time from the system, you can ask the time from the clock object.

> The primary purpose of this abstraction is to allow alternate clocks to be plugged in as and when required. Applications use an object to obtain the current time rather than a static method. This can simplify testing.

During testing you can use the `Clock.fixed` static method to get a clock that always reports a fixed time.

## Example

To make things more clear, let's have a look at a simple example of a class which has a replaceable clock.

{% highlight java %}
public class ReplaceableClockDemo {

    public static void main(String[] args) {
        ReplaceableClockDemo demo = new ReplaceableClockDemo(Clock.systemUTC());
        System.out.println(demo.isNowBefore(LocalDateTime.of(2017, 3, 1, 12, 0)));
    }

    private final Clock clock;

    public ReplaceableClockDemo(Clock clock) {
        this.clock = clock;
    }

    public boolean isNowBefore(LocalDateTime dateTime) {
        return LocalDateTime.now(clock).isBefore(dateTime);
    }

}
{% endhighlight %}

`ReplaceableClockDemo` has an instance field to hold a reference to a `Clock`. It can be passed in via the class' constructor. This allows the client to provide its own clock implementation. In the main method you can see how this class could be used in a production environment. When the class is instantiated, a reference to a system clock is provided.

In unit testing though, you're free to pass in a fixed clock. As a result, you have full control over the current time in your tests and you don't have to rely on static methods to set the current system clock.

{% highlight java %}
public class ReplaceableClockDemoTest {

    @Test
    public void shouldBeBeforeStPatricksDay2017() throws Exception {
        Clock now = Clock.fixed(LocalDateTime.of(2017, 2, 15, 12, 34)
                .toInstant(ZoneOffset.UTC), ZoneId.of("UTC"));
        LocalDateTime stPatricksDay = LocalDateTime.of(2017, 3, 17, 0, 0);
        ReplaceableClockDemo demo = new ReplaceableClockDemo(now);
        assertTrue(demo.isNowBefore(stPatricksDay));
    }

    @Test
    public void shouldNotBeBeforePiDay2017() throws Exception {
        Clock now = Clock.fixed(LocalDateTime.of(2017, 5, 18, 9, 45)
                .toInstant(ZoneOffset.UTC), ZoneId.of("UTC"));
        LocalDateTime piDay = LocalDateTime.of(2017, 3, 14, 0, 0);
        ReplaceableClockDemo demo = new ReplaceableClockDemo(now);
        assertFalse(demo.isNowBefore(piDay));
    }
}
{% endhighlight %}

## What about setting the current time via a static method?

If you've ever used [Joda-Time](http://www.joda.org/joda-time/ "Joda-Time"), then you're probably familiar with the `DateTimeUtils.setCurrentMillisFixed()` static method. It sets the current time but it does this globally. Whenever `currentTimeMillis()` is queried, the same fixed millisecond time will be returned.

In theory, this approach works but it has some flaws in my opinion. First of all, [you need to make sure you reset the time after each test]({{site.url}}/articles/using-joda-time-in-unit-tests/ "Using Joda-Time in unit tests"). Otherwise, some tests that come afterwards might use the same static resource and get a time they did not expect to receive, resulting in test failures. You should not depend on the order of your tests to always be the same. Tests, that are independent are easier to maintain and you also have the added benefit of being able to run them in parallel.

## Summary

Coupling your code tightly to the system's clock prevents you from writing reliable unit tests. According to the dependency inversion principle we should depend upon abstractions. Design your classes in a way that you can pass them a desired concrete implementation of time. This allows you to easily set a fixed time in unit tests.
