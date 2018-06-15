---
layout: post
title: "5 ways to create a Stream in Java 8"
excerpt: Java 8 introduced the Streams API which allows developers to declaratively manipulate collections. Here are five ways to create a Stream in Java 8.
modified: 2016-12-18 20:18:17 +0200
categories: articles
tags: [java, java 8, streams api]
image:
    feature: 2016-12-12-5-ways-to-create-a-stream-in-java-8/cover.jpg
    credit: Ales Krivec
    creditlink: https://unsplash.com/photos/tB-1h16ganU
comments: true
share: true
published: true
aging: true
---

Even the simplest programs use some kind of collection of elements. They are an essential part in programming. Be it arrays, lists, dictionaries or maps, they're used to store data so it could be easily accessed and manipulated. Depending on the use case, different data structures are chosen. Arrays are good for storing a sequence of elements. Key-value data can be stored in dictionaries or maps (some programming languages call them associative arrays).

Java 8 introduced the Streams API which allows developers to declaratively manipulate collections. It promotes the usage of functional style programming. Together with [lambda expressions]({{site.url}}/articles/java-8-lambda-expressions/ "Overview of lambda expressions in Java 8") and [method references]({{site.url}}/articles/four-types-of-method-references-in-java-8/ "4 types of method references in Java 8"), Streams make code more concise and readable. Additionally Streams allow to pipe multiple operations one after another. If you're familiar with Unix [command-line pipes](http://www.westwind.com/reference/os-x/commandline/pipes.html "Command-line pipes") then you might find composing stream operations simple to understand. To get started with Streams, you need a method to create them. Here are five ways to create a Stream in Java 8.

## Creating a Stream from a Collection

Probably the most obvious method is to create a Stream from a [Collection](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html "Collection Javadoc"). This is the interface that lists, sets, queues and the like implement. With the introduction of Java 8 and [*default methods*](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html "Default Methods in Java"), a method called `stream()` was added to the Collection interface. It returns a sequential Stream with the collection as its source. The following example shows this in action.

{% highlight java %}
List<Book> library = new ArrayList<>();
library.add(new Book("Alice's Adventures in Wonderland", "Lewis Carrol"));
Stream<Book> bookStream = library.stream();
{% endhighlight %}

## Creating a Stream from arrays

Arrays don't have the same convenient methods that Collections have. Therefore to create a stream from an array, you need to use the static method `stream()` in the `Arrays` class. Next, you'll see how to create a Stream of type String from an array of strings.

{% highlight java %}
String[] names = {"Lewis Carrol", "H.G. Wells", "Michael Ende"};
Stream<String> stream = Arrays.stream(names);
{% endhighlight %}

In addition to general Streams, the `Arrays` class contains overloaded methods for specialized streams as well. These are streams that can be used with Java's primitive types. Giving the `stream()` method an array of primitive types returns a specialized stream.

{% highlight java %}
int[] integers = {1, 4, 6, 2, 6, 3, 2};
IntStream stream = Arrays.stream(integers);
{% endhighlight %}

## Creating a Stream from values

Instead of creating an array or a collection and then converting it into a Stream, it is possible to create a Stream directly using the static method `of()` in the Stream interface.

{% highlight java %}
Stream<String> stream = Stream.of("Lewis Carrol", "H.G. Wells", "Michael Ende");
{% endhighlight %}

## Creating a Stream from files

Java 8 allows to create a Stream from a file. The `java.nio.file.Files` class contains several static methods which return a Stream of the file contents. The following example creates a Stream of Strings which represent the lines of the file.

{% highlight java %}
Stream<String> lines = Files.lines(Paths.get("/tmp/data"), Charset.defaultCharset());
{% endhighlight %}

Alternatively, if you're not interested in a particular file but rather in a directory in the file system, `Files.list()` method can create a lazily populated Stream of the elements that are the entries in the directory.

{% highlight java %}
Stream<Path> list = Files.list(Paths.get("/tmp/data"));
{% endhighlight %}

## Creating a stream from functions

Two static methods in the Stream interface allow you to create *infinite* Streams. Yes, that's right. Unlike a Collection or an array, a Stream can have no bounds. The methods we're going to look at are `iterate()` and `generate()`.

### Iterate

Iterate accepts an initial element and a function which is applied to the previous element to produce the next element. The following example produces an infinite Stream of even numbers.

{% highlight java %}
Stream<Integer> evenNumbers = Stream.iterate(0, n -> n + 2);
{% endhighlight %}

A [lambda expression]({{ site.url }}/articles/java-8-lambda-expressions/) of type [UnaryOperator](https://docs.oracle.com/javase/8/docs/api/java/util/function/UnaryOperator.html "UnaryOperator JavaDoc") is used to produce the next even number by adding 2 to the previous number.

### Generate

Generate accepts a [Supplier](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html "Supplier JavaDoc") to generate an infinite Stream of unsorted elements. This method is good for generating a Stream of random elements as can be seen in the following example.

{% highlight java %}
Stream<Double> randomNumbers = Stream.generate(Math::random);
{% endhighlight %}

## Conclusion

Learning the Streams API will help you to express complex ideas succinctly. The first step to getting started is creating a new stream object. We covered several methods of stream creation. In future articles we'll dive deeper into stream operations.
