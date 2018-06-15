---
layout: post
title: "Java 8 Puzzlers: Streams"
excerpt: In the spirit of the book Java Puzzlers by Joshua Bloch and Neal Gafter, this post discusses a Puzzler that can be found in Java 8 Streams.
modified: 2016-08-31 18:42:50 +0300
categories: articles
tags: [java, java 8, stream, puzzler]
image:
  feature: 2016-08-31-java-8-puzzlers-streams/cover.jpg
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

In the spirit of the book [Java Puzzlers](http://www.javapuzzlers.com/ "Java Puzzlers webpage") by Joshua Bloch and Neal Gafter I'm going to share a corner case in Java 8. I first found out about this by watching a [Virtual JUG](https://virtualjug.com/ "Virtual JUG") session titled [“Java 8 Puzzlers: The Strange, the Bizarre, and the Wonderful”](https://www.youtube.com/watch?v=lu4UTY940tg "Java 8 Puzzlers: The Strange, the Bizarre, and the Wonderful on Youtube") presented by [Baruch Sadogursky](https://twitter.com/jbaruch) and [Viktor Gamov](https://twitter.com/gamussa).

## Broken Eggs Tale

The puzzler that I have in mind is called “Broken Eggs Tale” and the story is the following. You create a shopping list. You add milk, bread and sausages to it before going to the grocery store. Then you recall you need eggs as well and add them to the list.

{% highlight java %}
List<String> list = new ArrayList<>();
list.add("milk");
list.add("bread");
list.add("sausage");
Stream<String> stream = list.stream();
list.add("eggs, don't forget eggs!");
stream.forEach(System.out::println);
{% endhighlight %}

Can you guess what the output will be? As you can see, the [stream]({{site.url}}/articles/5-ways-to-create-a-stream-in-java-8/ "5 ways to create a Stream in Java 8") is created before the final item is added to the list. Then the contents of the stream are printed out using a [method reference]({{site.url}}/articles/four-types-of-method-references-in-java-8/ "Method references in Java") to `System.out::println`.

**I challenge you to guess the output before reading further.**

One might think that "milk", "bread" and "sausage" will be printed out since the creation of the stream is called before adding "eggs" to the list. On the other hand Java 8 Streams are lazy. This means that nothing really happens until a terminal operation (in this case `forEach()`) is called, suggesting that all four list elements will be printed out.

Running the program reveals that all list elements are printed out. If you're experienced with Streams in Java then this puzzler should have been pretty easy for you.

## Don't eat sausages

Let's have a look at the following puzzler which is, in fact, a continuation of the previous puzzle.

{% highlight java %}
List<String> list = new ArrayList<>();
list.add("milk");
list.add("bread");
list.add("sausage");
list = list.subList(0, 2); //remove sausage
Stream<String> stream = list.stream();
list.add("eggs, don't forget eggs!");
stream.forEach(System.out::println);
{% endhighlight %}

The code remains mostly the same but "sausage" has been removed from the list. The question is still the same. What will be printed out? **Take a moment to think about it**.

From the previous puzzler, we know already that Streams are lazy. Taking that into account, all list items except "sausage" should be printed out. Seems like a fair guess, right? But running the code reveals something interesting. First two list items are printed out but then a `java.util.ConcurrentModificationException` is thrown. Okay, what are we missing here?

The obvious and *"correct"* answer should be that all list elements except "sausage" are printed out. But Java 8 has a [bug](https://bugs.openjdk.java.net/browse/JDK-8148748 "JDK-8148748") which prevents that from happening. This affects only ArrayLists. For instance, changing the list type to LinkedList fixes the problem. The bug has been fixed in Java 9 but unfortunately has not been backported to Java 8.

## Summary

Puzzlers are definitely interesting and I encourage you to read the book ["Java Puzzlers"](http://www.javapuzzlers.com/ "Java Puzzlers webpage") for more mind-bending content. Also have a look at the [Virtual JUG](https://virtualjug.com/ "Virtual JUG") session titled [“Java 8 Puzzlers: The Strange, the Bizarre, and the Wonderful”](https://www.youtube.com/watch?v=lu4UTY940tg "Java 8 Puzzlers: The Strange, the Bizarre, and the Wonderful on YouTube").
