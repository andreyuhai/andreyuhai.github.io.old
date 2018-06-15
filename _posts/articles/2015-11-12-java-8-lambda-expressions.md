---
layout: post
title: "Java 8: Lambda expressions"
excerpt: In computer programming, an anonymous function (also function literal or lambda abstraction) is a function definition that is not bound to an identifier.
modified: 2015-11-12 18:17:27 +0200
categories: articles
tags: [java 8, java, lambda, functional programming]
image:
  feature: 2015-11-01-java-8-lambda-expressions/cover.jpg
  credit: SamReidn
  creditlink: https://www.reddit.com/r/wallpapers/comments/ydx2p/an_abstract_lambda_wallpaper/
comments: true
share: true
published: true
---

In computer programming, an anonymous function (also function literal or lambda abstraction) is a function definition that is not bound to an identifier ([Wikipedia](https://en.wikipedia.org/wiki/Anonymous_function "Anonymous function")). Many modern programming languages have had them for a long time. It's only Java that's a little late to the party. While being 20 years old, it has received lambda expressions relatively recently.

Before Java 8, Java developers could use anonymous inner classes to achieve something similar to lambda expressions. [In my last post]({{site.url}}/articles/java-8-behavior-parameterization "Java 8: Behavior parameterization") I showed that this is verbose and discourages the use of passing code as a parameter.

## Benefits

Technically lambdas do not help you do anything that you could not do prior to Java 8. Remember that anonymous inner classes can achieve the same result but with more boilerplate code. The benefit of lambdas is improved and more flexible code.

**Before** (prior to Java 8):

{% highlight java %}
Collections.sort(books, new Comparator<Book>() {
    @Override
    public int compare(Book o1, Book o2) {
        return o1.getAuthor().compareTo(o2.getAuthor());
    }
});
{% endhighlight %}

**After** (using Java 8):

{% highlight java %}
books.sort((Book b1, Book b2) -> b1.getAuthor().compareTo(b2.getAuthor()));
{% endhighlight %}

In this example I sorted books by their author. That can be achieved with one line using a lambda expression. Technically this example shows another feature that Java 8 introduced as well - **default methods**. But that is not the topic of this post and I'm going to cover them in the future.

Making use of **method references**, this line can be improved even more.

{% highlight java %}
books.sort(comparing(Book::getAuthor));
{% endhighlight %}

## Syntax

The syntax of a lambda expression in Java is relatively easy.

{% highlight java %}
(Book b1, Book b2) -> b1.getAuthor().compareTo(b2.getAuthor());
{% endhighlight %}

It consists of 3 parts:

* list of parameters
* arrow
* expression body

If you have used lambda expressions in other languages you can see that the syntax is very similar. When the expression body spans multiple lines, it must be enclosed in curly braces.

{% highlight java %}
(parameters) -> { statements; }
{% endhighlight %}

## Type checking and type inference

The Java compiler can deduce the signature of a lambda expression. Therefore you do not have to explicitly define what types are used as parameters of a lambda expression. Looking at the book sorting example, it is possible to omit the `Book` type declarations.

**With explicit type declarations**
{% highlight java %}
books.sort((Book b1, Book b2) -> b1.getAuthor().compareTo(b2.getAuthor()));
{% endhighlight %}

**Omitting type declarations**
{% highlight java %}
books.sort((b1, b2) -> b1.getAuthor().compareTo(b2.getAuthor()));
{% endhighlight %}

This improves code readability and is especially useful if there's more than 2 parameters passed to a lambda expression. If there's only one parameter used, then the enclosing parenthesis can be removed.

{% highlight java %}
filter(books, b -> "Lewis Carrol".equals(b.getAuthor()));
{% endhighlight %}

## Where can I use a lambda expression

I used a lambda expression where the method parameter list expected a `Comparator` object. In my [previous post]({{site.url}}/articles/java-8-behavior-parameterization "Java 8: Behavior parameterization") I gave an example of filtering a list of books and there I used a lambda expression where the method expected a `Predicate` object. So what's the common denominator here? What must be the type of the parameter so I can pass in a lambda expression?

You can use a lambda expression in the context of a functional interface. It is an interface with only one abstract method. `Comparator` specifies the `compare()` method and the `Predicate` from my [previous post]({{site.url}}/articles/java-8-behavior-parameterization "Java 8: Behavior parameterization") specified one method as well - `test()`. If you look at the Java 8 API and search for [*Comparator*](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html "Java 8 API - Comparator") you'll see that it contains a lot of static and default methods as well. But it still specifies only one abstract method. Lambda expression's signature must match the signature of the functional interface.

Java 8 includes a lot of common functional interfaces which can be applied to most use cases. Following are a few examples.

|`Consumer<T>`   	|T -> void  |Takes a parameter but returns nothing   	|
|`Function<T, R>` |T -> R   	|Takes an input and returns an output   	|
|`Supplier<T>`   	|() -> T   	|Takes no parameters but returns type T   |

Look into [`java.util.function` package](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html "java.util.function package") for a longer list of available functional interfaces.

## @FunctionalInterface

`@FunctionalInterface` is an annotation which indicates that an interface is intended to be used as a functional interface. Additionally the compiler will give a helpful error if the interface defines more than one abstract method.

---

Programming languages evolve to meet the requirements of programmers. Some might say that it took Java language designers ages to add lambda expressions - a feature which has been present in many modern languages for many years. Better late than never. All in all, I think lambdas are a great addition to the Java language and together with the [Streams API]({{site.url}}/articles/5-ways-to-create-a-stream-in-java-8/ "5 ways to create a Stream in Java 8"), they will greatly improve code readability and developer happiness.
