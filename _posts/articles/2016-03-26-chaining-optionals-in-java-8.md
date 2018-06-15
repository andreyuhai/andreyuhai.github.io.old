---
layout: post
title: "Chaining Optionals in Java 8"
excerpt: Optionals in Java can help you to write a program without null checks. Chaining Optionals in a Stream enables to pick the first Optional which has a value and return it.
modified: 2018-01-20 19:50:50 +0200
categories: articles
tags: [java, java 8, optional, lambda, functional programming]
image:
  feature: 2016-03-26-chaining-optionals-in-java-8/cover.jpg
  credit: http://www.birthinjuryjustice.org/chain-command/
  creditlink: http://www.birthinjuryjustice.org/wp-content/uploads/2014/11/Chain.jpg
comments: true
share: true
published: true
---
[Optionals in Java]({{site_url}}/articles/optionals-in-java-8/ "What are Optionals?") can help you to write a program without *null checks*. In addition they can make your code more concise and easier to read. Let's look at an example how chaining Optionals in a *Stream* can help you **avoid unnecessary *null checks***.

## Motivation

The purpose of chaining Optionals in a [Stream]({{site.url}}/articles/5-ways-to-create-a-stream-in-java-8/ "5 ways to create a Stream in Java 8") is to pick the first Optional which has a value and return it. You could have multiple service methods which return Optionals. Passing them to a Stream as [method references]({{site_url}}/articles/four-types-of-method-references-in-java-8/ "Method references in Java 8") allows you to **process them lazily and return as soon as there's something to return**.

The following utility method takes *Suppliers* as arguments and returns the first Optional which has a value.
{% highlight java %}
public static <T> Optional<T> first(Supplier<Optional<T>>... suppliers) {
    return Arrays.asList(suppliers).stream()
            .map(Supplier::get)
            .flatMap(o -> o.map(Stream::of).orElseGet(Stream::empty))
            .findFirst();
}
{% endhighlight %}

It is possible to use [method references]({{site_url}}/articles/four-types-of-method-references-in-java-8/ "Method references in Java 8") or [lambda expressions]({{site_url}}/articles/java-8-lambda-expressions/ "Lambda expressions in Java 8") as arguments.

{% highlight java %}
first(this::find1, this::find2, this::find3);
{% endhighlight %}

## Improvements in JDK9

JDK8 does not provide a convenient API for creating a Stream out of an Optional. Apparently this is [fixed in JDK9](https://bugs.openjdk.java.net/browse/JDK-8050820 "OpenJDK issue tracker") and the utility method could be written as follows.

{% highlight java %}
public static <T> Optional<T> first(Supplier<Optional<T>>... suppliers) {
return Arrays.asList(suppliers).stream()
        .map(Supplier::get)
        .flatMap(Optional::stream)
        .findFirst();
{% endhighlight %}

Notice how the `flatmap` method has changed.
[JDK9 introduced additional methods to the `Optional` class that are worth looking at]({{site.url}}/articles/improvements-to-optional-in-java-9/ "Improvements to Optional in Java 9") and are not covered in this post.

## Food for thought

Think about how to implement the same concept without using Optionals or lambda expressions. It would probably include multiple null checks. Using a functional style can help you implement a more general solution which is hopefully easier to understand as well.

This blog post was inspired by a question and answers  on [StackOverflow](http://stackoverflow.com/a/34398891/2928051).
