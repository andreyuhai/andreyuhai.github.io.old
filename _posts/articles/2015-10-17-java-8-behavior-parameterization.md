---
layout: post
title: "Java 8: Behavior parameterization"
excerpt: Behavior parameterization is essentially taking a block of code and making it available without executing it. Here's how to do it with Java.
modified: 2015-10-17 21:55:09 +0300
categories: articles
tags: [java 8, java, predicate, function, streams, lambda, lambdaj, guava, retrolambda]
image:
  feature: 2015-10-17-java-8-behavior-parameterization/cover.jpg
  credit: http://xtom.deviantart.com/
  creditlink: http://xtom.deviantart.com/art/Lambda-28926886
comments: true
share: true
published: true
---

Behavior parameterization is essentially taking a block of code and making it available without executing it. It can be passed to a method for example. **Since Java 8 introduced lambdas** (finally), it is now possible to parameterize method's behavior with anonymous functions. If you're familiar with Javascript, Scala, Groovy or any other language that has lambdas built in, then you probably have been using behavior parameterization all along.

It might not come to you as a surprise that in software development user requirements can change. Passing behavior as a parameter can help relieve the pain of change.

Unfortunately there are applications which are not going to be upgraded to run with the latest release of Java. Therefore I'm going to cover alternative solutions which can be used on runtimes previous to Java 8. In this post I will start with examples on how to achieve behavior parameterization with previous Java versions and then compare these solutions to lambdas. In the process I'm trying to show how idioms from functional programming can make your life easier as a software developer.

## TL;DR

If you wish to see the end result, check out [this Github Gist](https://gist.github.com/indrekots/d2b786ec2d5a9d9e66b4 "link to super short overview") which tries to carry the essence of this post in code.

## Example domain

Let's look at an example of filtering Java objects. More specifically, I'm going to use Java 7 to filter a list of `book` objects without using any external libraries. The book class has 3 fields: `name`, `pageCount` and `author`. Imagine we have a library application and according to requirements it should be possible to find books that have more than 200 pages.

{% highlight java %}
public class Book {
    private String name;
    private int pageCount;
    private String author;

    //getters and setters omitted...
}
{% endhighlight %}

The easiest way to find long novels (in our case a long novel is a book with more than 200 pages) is to loop over the list of books, use an if clause to check if it has more than the specified amount of pages, add the book to the result list and finally return it.

{% highlight java %}
public static List<Book> findLongNovels(List<Book> books) {
    List<Book> result = new ArrayList<>();
    for (Book book : books) {
        if (book.getPageCount() > 200) {
            result.add(book);
        }
    }
    return result;
}
{% endhighlight %}

## Changing requirements

Everything's fine and dandy, right? As is customary, requirements change and new requirements are added. Now the library application should be able to filter books by author. Pretty simple to accomplish. Just use the same general layout as before.

{% highlight java %}
public static List<Book> filterBooksByAuthor(List<Book> books, String author) {
    List<Book> result = new ArrayList<>();
    for (Book book : books) {
        if (author.equals(book.getAuthor())) {
            result.add(book);
        }
    }
    return result;
}
{% endhighlight %}

## Code reuse

If you compare `filterBooksByAuthor` to the previous `findLongNovels` method, you can clearly see that they're very similar. This is a [WET](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself "Wikipedia DRY vs WET") solution. Let's [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself "Wikipedia DRY vs WET") it up. The overall structure is the same. Code iterates over a list of books and applies a filtering clause. The goal is to keep the iteration and filtering separate. When using Java 7, we could create a `BookPredicate` interface which could define the logic for filtering. A [predicate](https://en.wikipedia.org/wiki/Predicate_(mathematical_logic) "Wikipedia: Predicate (mathematical logic)") is essentially a boolean-valued function. Since Java 7 does not have lambdas, we're going to wrap the predicate in a class.

{% highlight java %}
public interface BookPredicate {
    boolean test(Book book);
}
{% endhighlight %}

Filtering logic can be moved to a separate class which implements the `BookPredicate` interface.

{% highlight java %}
public class LengthPredicate implements BookPredicate {
    private int length;

    public LengthPredicate(int length) {
        this.length = length;
    }

    @Override
    public boolean test(Book book) {
        return book.getPageCount() > length;
    }
}

public class AuthorPredicate implements BookPredicate {
    private String author;

    public AuthorPredicate(String author) {
        this.author = author;
    }

    @Override
    public boolean test(Book book) {
        return author.equals(book.getAuthor());
    }
}
{% endhighlight %}

After doing some refactoring, it is possible to reuse the method that iterates over the list of books.

{% highlight java %}
public static List<Book> filterBooks(List<Book> books, BookPredicate p) {
    List<Book> result = new ArrayList<>();
    for (Book book : books) {
        if (p.test(book)) {
            result.add(book);
        }
    }
    return result;
}

//somewhere in main
BookPredicate lengthPredicate = new LengthPredicate(200);
BookPredicate authorPredicate = new AuthorPredicate("Lewis Carrol");
filterBooks(books, lengthPredicate);
filterBooks(books, authorPredicate);
{% endhighlight %}

## Too verbose?

Now we're not repeating ourselves but hey, that's a lot of code to write. As they say, **Java is verbose**. Initially there were 2 methods that filtered books. That's about 15 lines of code. After removing duplicate code and moving filtering logic to separate classes there's more than 30 lines. Although this is not much for a small project, with a large project the lines add up. Is there anything that can be done to write more concise code?

## Anonymous inner classes

Instead of defining a concrete implementation of a `BookPredicate`, let's create one on the fly.

{% highlight java %}
filterBooks(books, new BookPredicate() {
    @Override
    public boolean test(Book book) {
        return "Lewis Carrol".equals(book.getAuthor());
    }
});
{% endhighlight %}

That's pretty concise. It looks almost like a lambda. As a matter of fact, when using Java 8, IDEs will suggest you to replace it with a lambda.  The downside of an anonymous inner class is that it comes with boilerplate code. A new object needs to be instantiated, a method is needed to be overridden and some curly braces here and there. That boilerplate makes it harder to focus on the part that's actually important - the comparison inside the `test` method.

## Using third party libraries

As expected, libraries are created to overcome the shortcomings of a language. Uncle Bob writes in his [blog post](http://blog.8thlight.com/uncle-bob/2015/08/06/let-the-magic-die.html "Make the Magic go away by Uncle Bob") that we write frameworks to compensate for the lack of features that we wish were in our language. Every framework you've ever seen is really just an echo of this statement:

>My language sucks!

What alternatives are out there then? [Google Guava](https://github.com/google/guava "Google Guava github repository") library has predicates which allow you to do more functional style programming.

{% highlight java %}
Iterables.filter(books, new Predicate<Book>() {
    @Override
    public boolean apply(Book input) {
        return "Lewis Carrol".equals(input.getAuthor());
    }
});
{% endhighlight %}

It is very similar to what we implemented with `filterBooks` method. In functional programming, filtering a list of items is done by applying a predicate to each element of the list. `Filter` is a common function in functional languages. Later we'll see that Java 8 has included it as well. The benefits of using Guava is that you do not have to write list iteration code and the predicate interface.

Another possible solution is to use [lambdaj](https://github.com/mariofusco/lambdaj "lambdaj github repository") with [Hamcrest](https://github.com/hamcrest/JavaHamcrest "Hamcrest github repository") matchers. lambdaj is a library that allows you to manipulate collections in a pseudo-functional and statically typed way.

{% highlight java %}
filter(having(on(Book.class).getAuthor(), equalTo("Lewis Carrol")), books);
{% endhighlight%}

Wow, all that in one line. This will get a bit messier if there's a more complicated filtering clause though.

## Java 8 lambdas

The latest release brings some new features which will improve code readability and help the language stay competitive in the future. Let's look at the book filtering example and see how behavior parameterization can be used with lambdas built into the language.

First we need to rewrite the `filterBooks` method to use `java.util.function.Predicate` which is a new interface in Java 8.

{% highlight java %}
public static List<Book> filterBooks(List<Book> books, Predicate<Book> p) {
    List<Book> result = new ArrayList<>();
    for (Book book : books) {
        if (p.test(book)) {
            result.add(book);
        }
    }
    return result;
}
{% endhighlight %}

And when calling `filterBooks`, we can pass it a lambda expression which tells it how to do the filtering.

{% highlight java %}
filterBooks(books, book -> "Lewis Carrol".equals(book.getAuthor()));
{% endhighlight %}

Although we used a lambda expression and made `filterBooks` method's behavior parameterizable, there is still this boilerplate code which iterates over a list of books. Previously I mentioned that Java 8 has included the filter idiom which is common among functional languages. [Streams]({{site.url}}/articles/5-ways-to-create-a-stream-in-java-8/ "5 ways to create a Stream in Java 8") is a new API which helps to express sophisticated data processing queries. Among others, it includes a filter method.

{% highlight java %}
books.stream().filter(b -> "Lewis Carrol".equals(b.getAuthor())).collect(toList());
{% endhighlight %}

As it can be seen, the list of books is not passed to a method but we can call the `filter` method on it by first creating a stream from it. Iteration is handled by the [Streams API]({{site.url}}/articles/5-ways-to-create-a-stream-in-java-8/ "5 ways to create a Stream in Java 8") and behavior is parameterizable thanks to lambdas. So instead of writing a lot of boilerplate code, Java 8 takes care of the commonly occurring tasks and you can solve the problem at hand with just one line of code.

## Remember those changing requirements?

In the beginning of this post I gave an example of changing requirements. Now that lambdas can be used, let's see how the library application can handle a new feature request. It should be possible to find books that have more than 200 pages.

{% highlight java %}
books.stream().filter(b -> b.getPageCount() > 200).collect(toList());
{% endhighlight %}

Without modifying any existing code, it is very easy to filter the list of books with a new behavior.

## Retrolambda

If you're using previous versions of Java then you can still take advantage of lambdas by using [Retrolambda](https://github.com/orfjackal/retrolambda "Retrolambda Github repository"). It lets you run Java 8 code with **lambda expressions**, **method references** and **try-with-resources statements** on Java 7, 6 or 5. It does this by transforming your Java 8 compiled bytecode so that it can run on an older Java runtime. I'm not an expert of its inner workings but from what I've read, it replaces lambdas with anonymous inner classes.

Retrolambda does not backport the Streams API though. For that you could use [streamsupport](http://sourceforge.net/projects/streamsupport/ "streamsupport Sourceforge page").

## Summary

Using idioms common in functional programming can greatly improve the readability of the code. Behavior parameterization is great because it enables you to separate the code that iterates over a collection from the behavior that is applied to each element of the collection. This results in better code reuse and helps you write more flexible APIs.
