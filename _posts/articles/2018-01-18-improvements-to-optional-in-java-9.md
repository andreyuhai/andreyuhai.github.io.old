---
layout: post
title: "Improvements to Optional in Java 9"
excerpt: Java 9 introduced three new methods to the Optional class. This post covers what they do and how to use them.
modified: 2018-01-18 16:46:57 +0200
categories: articles
tags: [java, java 9, optional]
image:
  feature: 2018-01-14-improvements-to-optional-in-java-9/cover.jpg
  credit: Matija Mestrovic
  creditlink: https://unsplash.com/photos/D2rJ0RlDZ58
comments: true
share: true
published: true
aging: true
---
[Optionals were introduced to Java in version 8]({{site.url}}/articles/optionals-in-java-8/).
If used correctly, they can help you avoid unintentional `NullPointerException`s.
Essentially, an `Optional` is a container object which may or may not contain a non-null a value.
Combined with [lambda expressions]({{site.url}}/articles/java-8-lambda-expressions/), you can write concise code that avoids explicit `null` checks.

{% highlight java %}
Optional<Book> bookOptional = findBook("The War of the Worlds");
Book book = bookOptional.orElseGet(Book::defaultBook);
{% endhighlight %}

Java 9 introduced a couple of improvements to the [Optional](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html) class.

## `Stream` from `Optional`

[`Optional::stream`](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html#stream-- "Optional javadoc for Java 9") allows you to transform an `Optional` to a `Stream`.
If a value is present, the returned `Stream` will contain that value.
Otherwise an empty `Stream` is returned.
You might be thinking how can that be useful?
We use Streams to manipulate collections of data.
How can a `Stream` that contains at most one element be useful?

In a `Stream` processing pipeline we can transform `Optional`s into `Stream`s and `flatmap` them into a single `Stream` containing only the values that are present.
Let's look at the following example.

{% highlight java %}
List<Book> books = ...

List<Author> ghostwriters = books.stream()
  .map(Book::getGhostWriter)
  .flatMap(Optional::stream)
  .collect(toList());
{% endhighlight %}

Since every book doesn't have a [ghostwriter](https://en.wikipedia.org/wiki/Ghostwriter), `getGhostWriter()` returns an `Optional` of `Author` (`Optional<Author>`).
For each book, `Optional::stream` creates a new `Stream` and `flatmap` will ensure that all `Stream`s are merged together.
Essentially, we're getting rid of empty `Optional`s.

You could do it in Java 8 with a [lambda expression]({{site.url}}/articles/java-8-lambda-expressions/) but `Optional::stream` is more succinct.

{% highlight java %}
List<Book> books = ...

List<Author> ghostwriters = books.stream()
  .map(Book::getGhostWriter)
  .flatMap(o -> o.map(Stream::of).orElseGet(Stream::empty))
  .collect(toList());
{% endhighlight %}

## Optional::or

[`Optional::or`](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html#or-java.util.function.Supplier-) will either return the current `Optional` if a value is present.
Otherwise an `Optional` produced by the provided supplier function is returned.
To make things more clear, let's have a look at the following example.

{% highlight java %}
findBook().or(() -> findBookFromWeb());
{% endhighlight %}

If `findBook()` does not find anything, `findBookFromWeb()` gets called.
Since the return value is always an `Optional`, we can build a lazily loaded chain of method calls.

{% highlight java %}
public Optional<Book> findBook(String title) {
  return findBookFromDatabase(title)
    .or(() -> findBookFromLocalLibrary(title))
    .or(() -> findBookFromWeb(title));
}
{% endhighlight %}

Back in [Java 8 you had to go through some ceremony]({{site.url}}/articles/chaining-optionals-in-java-8/) to achieve something similar.

## Optional::ifPresentOrElse

The final addition to `Optional`s in Java 9 is the [`Optional::ifPresentOrElse`](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html#ifPresentOrElse-java.util.function.Consumer-java.lang.Runnable-) method.
By accepting a consumer function and a runnable, it can be used to cover the case where the value is present as well as the case where the `Optional` is empty.

{% highlight java %}
findBook(title).ifPresentOrElse(
    this::increaseReaderCount,
    () -> log.warn("Tried to look for a book which does not exist!"));
{% endhighlight %}

## Summary

Three new methods were added to the `Optional` class in Java 9. `Optional::stream` allows you to transform an `Optional` into a `Stream` of one element or an empty `Stream`. `Optional::or` returns the current `Optional` if a value is present, otherwise an `Optional` produced by the supplier function is returned. `Optional::ifPresentOrElse` can be used to cover the value present as well as the value missing cases.
