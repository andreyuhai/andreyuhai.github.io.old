---
layout: post
title: "Optionals in Java 8"
excerpt: If you have had any experience with Java, then you most likely have seen the NullPointerException. Optional in Java 8 are a way to fight them.
modified: 2018-01-20 07:16:38 +0200
categories: articles
tags: [java,java 8,optional,lambda,null reference]
image:
  feature: 2016-03-12-optionals-in-java-8/cover.jpg
  credit: Sebastian Unrau
  creditlink: https://unsplash.com/photos/CoD2Q92UaEg
comments: true
share: true
published: true
---

## Problems with null

If you have had any experience with Java, then you probably have seen the *NullPointerException* which is thrown when an application tries to use a null reference in a case where an object is required. This can lead to **superfluous if-statements** checking to see if a reference is null or not.

Null references were implemented because they were the easiest method to implement the absence of a value. Tony Hoare, a British computer scientist, designer of the [ALGOL W programming language](https://en.wikipedia.org/wiki/ALGOL_W "Wikipedia page of ALGOL W") and the inventor of null references called it his [billion-dollar mistake](https://www.lucidchart.com/techblog/2015/08/31/the-worst-mistake-of-computer-science/ "Blog post about the worst mistake of computer science").

>I call it my billion-dollar mistake. It was the invention of the null reference in 1965. At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W). My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years. - Tony Hoare

## How to avoid NullPointerExceptions

The easiest answer is that you should never return a *null*. But that's easier said than done. What should I return when there is no value to return? What if the library I'm using returns nulls?

Apparently null reference is not the best way to model an absence of a value. Instead you should return an object which represents an absence of a value. But wouldn't that lead to superfluous if-else statements checking if this object contains a value or not? Bear with me until I show how Optionals can solve this problem. But first, let's quickly go over how other languages have solved the same issue.

A lot of other languages have introduced a *Maybe* type - something that represents a sate where there might not be a value. For example, Scala has `scala.Option` and Standard ML uses `option`. Even if the language does not provide a concept of *Maybe*, there usually is a library that solves the problem. For instance, [Guava, the popular Java library by Google](https://github.com/google/guava "Guava Github page"), includes an [Optional class](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Optional.html Guava Optional javadoc).

## Optional<T>

Optional is a container object which may or may not contain a non-null value. It has many useful methods that make programming without null checks more convenient. Let's look at a few examples.

The example domain contains books. Creating an optional is easy. The class provides static methods to do so.
{% highlight java %}
//create an empty optional
Optional<Book> empty = Optional.empty();
//create an instance of a Book and wrap it inside an Optional
Optional<Book> bookOptional = Optional.of(new Book());
{% endhighlight %}

To get the contents of the *Optional* container, you can call the `get()` method on it. If you're unsure if the container contains a non-null value, it is possible to use the `isPresent()` method. It returns `true` if it contains a non-null value. Be aware that checking the container for contents and then retrieving it **defeats the purpose of *Optionals* in my opinion**. In terms of superfluous if-statements, it is the same as checking if an object is null or not.

Instead I would advise you to look at the methods provided by the [Optional class](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html "Java Optional javadoc page") and see if you can come up with a more clever solution. The following are a few useful use cases where Optional is used.

## Examples of Optional

Let's look at some examples of Optionals in action. As previously mentioned, calling the `get()` method on an Optional, will return its contents. Instead of calling `get()` it's better to use the `orElse()` method to which you can pass a value that will be returned if the Optional contains a null reference.

{% highlight java %}
Optional<Book> bookOptional = findBook("The War of the Worlds");
Book book = bookOptional.orElse(new Book());
{% endhighlight %}

If the Optional contains a null reference, a new book is returned. Depending on the situation, returning a plain Book object might not desirable since all of it's fields are empty. The API provides other useful methods as well.

{% highlight java %}
Optional<Book> bookOptional = findBook("The War of the Worlds");
Book book = bookOptional.orElseGet(Book::defaultBook);
{% endhighlight %}

The `orElseGet()` method expects a Supplier which you can implement with a [lambda expression]({{site_url}}/articles/java-8-lambda-expressions/). In this example I provided a [method reference]({{site_url}}/articles/four-types-of-method-references-in-java-8/) which returns a default book.

With the `ifPresent()` method it is possible to [execute a behavior]({{site_url}}/articles/java-8-behavior-parameterization/) if the optional contains a value.

{% highlight java %}
bookOptional.ifPresent(System.out::println);
{% endhighlight %}

Rather than retrieving a Book object, you can extract the instance fields from the Book object inside the Optional container. In the following example we get the name of the book. If the name is not present or there's no book at all, a default value is returned.
{% highlight java %}
String name = bookOptional.map(Book::getName).orElse("Name not provided");
{% endhighlight %}

The same method can be used if there's a chain of objects.

{% highlight java %}
String name = bookOptional.map(Book::getPublisher).map(Publisher::getName).orElse("Unknown publisher");
{% endhighlight %}

If there's a null reference in the chain, "Unknown publisher" is returned.

## Other useful methods

I don't intend to write about all the API methods. But just so you know, there are other useful methods that might interest a Java developer (e.g. `filter()` and `flatMap()`). I'd strongly encourage you to look at the [Javadoc for the *Optional* class](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html#get-- "Optional javadoc page").

## Summary

If you're used to checking if a value is null then Optionals require a slightly different mindset. With the support of [lambda expressions]({{site_url}}/articles/java-8-lambda-expressions/) it is possible to avoid null checks unless you're using an older Java library which does not support Optionals. The solution here would be to create a wrapper but it's up to you to decide if the effort is worth it. Optionals are not serializable and are not meant to be used as field types. Therefore if you plan to use Optionals in your domain model, I would use the actual types as field types and return Optionals from getters. This can provide you with better compatibility with ORM frameworks.

If you want to learn more about `Optional`s, you should have a look at what [new features were added to the `Optional` class in Java 9]({{site.url}}/articles/improvements-to-optional-in-java-9/ "Improvements to Optional in Java 9").
