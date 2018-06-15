---
layout: post
title: "Four types of method references in Java 8"
excerpt: Method references let you reuse existing method definitions and pass them just like lambdas. They can be useful in making the code more readable.
modified: 2015-11-28 20:55:38 +0200
categories: articles
tags: [java, java 8, method reference, lambda, class, constructor]
image:
  feature: 2015-11-28-four-types-of-method-references-in-java-8/cover.jpg
  credit:
  creditlink:
comments: true
share: true
published: true
---

Method references let you reuse existing method definitions and pass them just like lambdas. They can be useful in making the code more readable. Instead of writing a full lambda expression, it is possible to point to an existing method.

Java 8 includes four types of method references:

* A method reference to a static method
* A method reference to an instance method of an arbitrary type
* A method reference to an instance method of an existing object
* Constructor reference

Let's go over each type.

## A method reference to a static method

This is probably the easiest to explain with the following example:

{% highlight java %}
Class::staticMethod
{% endhighlight %}

Rather than using `s -> Integer.parseInt(s)`, the same can be achieved with `Integer::parseInt`.

## A method reference to an instance method of an arbitrary type

The idea with this type of method reference is that you’re referring to a method of an object that will be supplied as one of the parameters of the lambda.

For example
{% highlight java %}
(String s) -> s.toUpperCase()
{% endhighlight %}

can be written as

{% highlight java %}
String::toUpperCase
{% endhighlight %}

## A method reference to an instance method of an existing object

With this method reference type you can refer to a method of an existing object. For example you might have an instance variable `books` which is of type `List<Book>`. Instead of referring to the size of the list with a full lambda expression

{% highlight java %}
() -> books.size()
{% endhighlight %}

you could use a method reference

{% highlight java %}
books::size
{% endhighlight %}

## Constructor reference

It is possible to reference a constructor with the following syntax `Class::new`. It works similarly to the reference to a static method. Continuing with the book theme, you can reference a no arguments constructor of a `Book` class

{% highlight java %}
Book::new
{% endhighlight %}

But what if I want to use a constructor with arguments. A [lambda expression/method reference]({{site.url}}/articles/java-8-lambda-expressions/ "Java 8: Lambda Expressions") can be used in a context of a functional interface. The type of a no-arg constructor fits the signature `() -> Book` and matches the type `Supplier<Book>`.

The signature of the `Book` constructor with one argument is `Book(String name)`. To use the one argument constructor as a method reference, you should use the `Function` interface.

{% highlight java %}
Function<String, Book> b = Book::new;
Book book = b.apply("All Quiet on the Western Front");
{% endhighlight %}

## Special forms of method references

**Reference to an array constructor**

Arrays can be constructed with the `new` keyword.

{% highlight java %}
int[] array = new int[10];
{% endhighlight %}

This can be rewritten with a method reference as follows

{% highlight java %}
IntFunction<int[]> c = int[]::new;
int[] array = c.apply(10);
{% endhighlight %}

**This and super in method references**

Methods can be referenced with the help of the `this` and `super` keyword. In the following example `this` is used in the process of creating a thread.

{% highlight java %}
//instance method in class
void sayHello() {
  System.out.println("Hello World!");
}

//somewhere in the class
new Thread(this::sayHello).start();
{% endhighlight %}

The `super` keyword can be used the same way. Imagine that the `sayHello()` method is in a superclass. Then creating a thread which uses the parent class method as a Runnable is as follows

{% highlight java %}
new Thread(super::sayHello).start();
{% endhighlight %}

## Conclusion

In a nutshell, you can think of method references as syntactic sugar for lambdas. Using them can help the code gain readability. Short, one line lambdas are easy to read, but if your code or the Java API provides a method which you can refer to, then it's better to use a method reference. Deciding if it is worth extracting a long and complex lambda to a method is up to you. It depends on the context and I think there's no definitive answer.
