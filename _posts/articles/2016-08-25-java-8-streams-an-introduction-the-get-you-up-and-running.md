---
layout: post
title: "Java 8 Streams: an introduction the get you up and running"
excerpt:
modified: 2016-08-25 14:39:19 +0300
categories: articles
tags: [java, java 8, streams]
image:
  feature: 2016-08-25-java-8-streams-an-introduction-the-get-you-up-and-running/cover.jpg
  credit: Ales Krivec
  creditlink: https://unsplash.com/photos/tB-1h16ganU
comments: true
share: true
published: false
aging: true
---
* Table of Contents
{:toc}

Even the simplest programs use some kind of collection of elements. They are an essential part in programming. Be it arrays, lists, dictionaries or maps, they're used to store data so it could be easily accessed. Depending on the use case, different data structures are chosen. Arrays are good for storing a sequence of elements. Key-value data can be stored in dictionaries or maps (some programming languages call them associative arrays). Up until Java 8, processing Collections in Java has been inconvenient.

I must admit that some operations are relatively simple to achieve but if you're faced with a task to filter and then group data, you're probably going to write several lines of code with multiple intermediary data containers. It would be better to declaratively define what operations you want to be applied to the collection. Think of SQL. You can select, filter and group data from a table declaratively. Java designers introduced the Streams API which allows to manipulate collections in a declarative and functional manner.

## Streams

Streams are a new API in Java 8 which allows developers to declaratively manipulate collections of data. It promotes the usage of functional style programming. Together with lambda expressions, Streams make the code more concise and readable. Additionally Streams allow to pipe multiple operations one after another. If you're familiar with Unix command-line pipes then you might find composing stream operations simple to understand. All in all, it helps the code to be more composable.

I think the best way to demonstrate the conciseness of the Streams API is to show the following comparison. The first snippet uses Java 7 and the next uses Java 8 Streams to accomplish the same task.

{% highlight java %}
List<Book> library = new ArrayList<>();
//initialize a library of books
List<Book> library = new ArrayList<>();
library.add(new Book("Alice's Adventures in Wonderland", 300, "Lewis Carrol"));
library.add(new Book("Through the Looking-Glass, and What Alice Found There", 228, "Lewis Carrol"));
library.add(new Book("The War of the Worlds", 192, "H.G. Wells"));
library.add(new Book("1984", 268, "George Orwell"));
library.add(new Book("Animal Farm", 102, "George Orwell"));
library.add(new Book("The Neverending Story", 396, "Michael Ende"));

//group books by author using Java 7
Map<String, List<Book>> byAuthor = new HashMap<>();
for (Book book : library) {
    if (byAuthor.get(book.getAuthor()) == null) {
        List<Book> books = new ArrayList<>();
        books.add(book);
        byAuthor.put(book.getAuthor(), books);
    }
    else {
        List<Book> books = byAuthor.get(book.getAuthor());
        books.add(book);  
    }
}

System.out.println(byAuthor);

//group books by author using Java 8 Streams API
Map<String, List<Book>> byAuthor2 = library.stream().
        collect(groupingBy(Book::getAuthor));

System.out.println(byAuthor2);

//prints out:
//{Michael Ende=[Name: The Neverending Story, page count: 396, author: Michael Ende], H.G. Wells=[Name: The War of the Worlds, page count: 192, author: H.G. Wells], George Orwell=[Name: 1984, page count: 268, author: George Orwell, Name: Animal Farm, page count: 102, author: George Orwell], Lewis Carrol=[Name: Alice's Adventures in Wonderland, page count: 300, author: Lewis Carrol, Name: Through the Looking-Glass, and What Alice Found There, page count: 228, author: Lewis Carrol]}
//{Michael Ende=[Name: The Neverending Story, page count: 396, author: Michael Ende], H.G. Wells=[Name: The War of the Worlds, page count: 192, author: H.G. Wells], George Orwell=[Name: 1984, page count: 268, author: George Orwell, Name: Animal Farm, page count: 102, author: George Orwell], Lewis Carrol=[Name: Alice's Adventures in Wonderland, page count: 300, author: Lewis Carrol, Name: Through the Looking-Glass, and What Alice Found There, page count: 228, author: Lewis Carrol]}
{% endhighlight %}

Although they produce the same result (they group books by authors), using Streams is easier to understand and required significantly less work from the developer. Don't worry if you don't understand the code. I'll go over the basics in the following paragraphs.

## Creating a Stream

To get started with Streams, you need a method to create them. Java 8 provides several approaches and we'll have a look at them one by one.

### Creating a Stream from a collection

Collection is the interface which lists, sets, queues and the like implement. With the introduction of Java 8 and *default methods*, a method called `stream()` was added to the Collection interface. It returns a sequential Stream with the collection as its source.

{% highlight java %}
List<Book> library = new ArrayList<>();
library.add(new Book("Alice's Adventures in Wonderland", 300, "Lewis Carrol"));
Stream<Book> bookStream = library.stream();
{% endhighlight %}

### Creating a Stream from arrays

Arrays don't have the same convenient methods that Collections have. Therefore to create a stream from an array, you need to use the static method `stream()` in the Arrays class. In addition to general Streams, it contains overloaded methods for specialized streams as well. Specialized streams are discussed later in this post.

{% highlight java %}
String[] names = {"Lewis Carrol", "H.G. Wells", "Michael Ende"};
Stream<String> stream = Arrays.stream(names);
{% endhighlight %}

### Creating a Stream from values

Instead of creating an array or a collection and then converting it into a Stream, it is possible to create a Stream directly using the static method `of()` in the Stream interface.

{% highlight java %}
Stream<String> stream = Stream.of("Lewis Carrol", "H.G. Wells", "Michael Ende");
{% endhighlight %}

### Creating a Stream from files

Java 8 allows to create a Stream from a file. The `java.nio.file.Files` class contains several static methods which return a Stream of the file contents. The following example creates a Stream of Strings which represent the lines of the file.

{% highlight java %}
Stream<String> lines = Files.lines(Paths.get("/tmp/data"), Charset.defaultCharset());
{% endhighlight %}

### Creating a stream from functions

Two static methods in the Stream interface allow you to create *infinite* Streams. Yes, that's right. Unlike a Collection or an array, a Stream can have no bounds. These methods are `iterate()` and `generate()`.

#### Iterate

Iterate accepts an initial element and a function which is applied to the previous element to produce the next element. The following example produces an infinite Stream of even numbers.

{% highlight java %}
Stream<Integer> evenNumbers = Stream.iterate(0, n -> n + 2);
{% endhighlight %}

A [lambda expression]({{ site.url }}/articles/java-8-lambda-expressions/) of type [UnaryOperator](https://docs.oracle.com/javase/8/docs/api/java/util/function/UnaryOperator.html "UnaryOperator JavaDoc") is used to produce the next even number by adding 2 to the previous number.

#### Generate

Generate accepts a [Supplier](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html "Supplier JavaDoc") to generate an infinite Stream of unsorted elements. This method is good for generating a Stream of random elements as can be seen in the following example.

{% highlight java %}
Stream<Double> randomNumbers = Stream.generate(Math::random);
{% endhighlight %}

## Stream operations

The Stream interface defines operations that can be classified into two broad categories - intermediate and terminal operations. Intermediate operations return another stream. This makes it possible to chain multiple operations together to form a query. Terminal operations consume the stream and process the intermediate operations. This means that Streams are lazy. Intermediate operations are processed only when a terminal operation is invoked. Depending on the terminal operation used, the return value can be *void* or a non-stream value (Integer, List, Map etc.).

## Examples of Stream operations

Let's have a look at some of the most common stream operations and how to use them. Streams introduce many operations that are widely used in functional programming languages such as map, reduce and filter.

### Map

In many programming languages (especially functional programming languages) map is a [higher-order function](https://en.wikipedia.org/wiki/Higher-order_function "Wikipedia page for higher-order functions") that applies a given function to each element of a list. In the context of Java 8 Streams, map accepts a [Function](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html "Function interface Javadoc") and returns a Stream where the given function has been applied to each element of the Stream.

Although we have not yet looked at terminal operations in more detail, I'm using the `forEach()` terminal operation in the following example to illustrate what the Stream elements look like. The `library` variable is the same list that was created in the first code example of this post.

{% highlight java %}
//library is a list containing Book objects
library.stream().map(Book::getName).forEach(System.out::println);

//prints out:
//Alice's Adventures in Wonderland
//Through the Looking-Glass, and What Alice Found There
//The War of the Worlds
//1984
//Animal Farm
//The Neverending Story
{% endhighlight %}

A [method reference]({{site.url}}/articles/four-types-of-method-references-in-java-8/) to `getName()` on the Book class is passed to map. The map operation returns a new Stream where Book objects have been replaced with Strings containing the book name.

//flatmap

### Reduce

Reduce is another higher-order function which is common in functional programming languages. Although the Java designers chose to use the name *reduce*, it is also widely known as *fold* or *accumulate*. Unlike map which returns a Stream, reduce returns a single value by applying an accumulation function to the elements of the Stream. Because a non-stream value is returned, reduce is classified as a terminal operation.

{% highlight java %}
String[] books = {"Alice's Adventures in Wonderland", "1984", "The Neverending Story"};
String result = Arrays.stream(books).reduce("", (a, b) -> a + " " + b);
System.out.println(result);

//prints out:
// Alice's Adventures in Wonderland 1984 The Neverending Story
{% endhighlight%}

Reduce can be used to concatenate strings. Although there is a better method to join a stream of strings, it perfectly illustrates how reduce works. It accepts an initial value which in this case is an empty string. If the stream contains no elements, then an empty string is returned. The second argument is a [BinaryOperator](https://docs.oracle.com/javase/8/docs/api/java/util/function/BinaryOperator.html "Javadoc for BinaryOperator") which takes the next element in the stream and concatenates it to the accumulated value. At first the accumulated value is an empty string which was passed as the first argument.

//create a diagram

The Java API specifies two more overloaded reduce methods. It is possible to omit the initial value and pass only a BinaryOperator to the reduce method. But in that case the return value is going to be an [Optional]({{ site.url }}/articles/optionals-in-java-8/) since there is no way of knowing what to return if there are no elements in the stream.  

{% highlight java %}
String[] books = {"Alice's Adventures in Wonderland", "1984", "The Neverending Story"};
Optional<String> result = Arrays.stream(books).reduce((a, b) -> a + " " + b);
result.ifPresent(System.out::println);

//prints out:
//Alice's Adventures in Wonderland 1984 The Neverending Story
{% endhighlight %}

The third and final overloaded reduce method is probably the most difficult to understand. It accepts an initial value called identity, a BiFunction and a BinaryOperator.

{% highlight java %}
<U> U reduce(U identity,
   BiFunction<U,? super T,U> accumulator,
   BinaryOperator<U> combiner)
{% endhighlight %}

The accumulator function is used to transform the next element in the stream of type T into type U which is then *accumulated* with the result of previous accumulations. In essence, it is a combined map and reduce operation. Most of the times it can be represented more simply by explicit calling map and reduce.

Although technically not needed in sequential streams, the combiner function combines two elements of type U into a single one. It is necessary to combine the intermediary results of a parallel stream (parallel streams are covered later in this post). The Streams API does not differ between parallel and sequential streams. Therefore in order to make reduce run correctly when executed parallely, it needs to know how to combine intermediate results.

//explain more with an example, SO link

{% highlight java %}
//library is a list of Book objects
String s = library.stream().reduce("", (s, b) -> s + ", " + b.getName(), (a, b) -> a + ", " + b);
System.out.println(s);

//prints out:
// Alice's Adventures in Wonderland Through the Looking-Glass, and What Alice Found There The War of the Worlds 1984 Animal Farm The Neverending Story
{% endhighlight %}

As stated previously, to improve the readability of this reduce operation, we could explicitly call map and then reduce.

{% highlight java %}
String s = library.stream().map(Book::getName).reduce("", (a, b) -> a + " " + b);
{% endhighlight %}

But why does the Streams API provide a method that could be represented better by pipelining multiple stream operations together? The answer is efficiency. [From Oracle docs](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#reduce-U-java.util.function.BiFunction-java.util.function.BinaryOperator- "Documentation for reduce") you can find the following:

>The accumulator function acts as a fused mapper and accumulator, which can sometimes be more efficient than separate mapping and reduction, such as when knowing the previously reduced value allows you to avoid some computation.

### Filter

As the name implies, *filter* is used to filtrate a Stream. It accepts a [predicate](https://en.wikipedia.org/wiki/Predicate_(mathematical_logic)) and returns a new Stream consisting of elements that match the given predicate.

Using the list of books that was created in the beginning of this post, I can find all book objects where the page count is greater than 300.

{% highlight java %}
library.stream().filter(b -> b.getPageCount() > 300).forEach(System.out::println);
//prints out:
//The Neverending Story
{% endhighlight %}

### Distinct

In a situation where you have duplicate elements in a Stream, you can use *distinct* to return a new Stream with only unique elements. The uniqueness is determined using the [*hashCode*](https://www.sitepoint.com/how-to-implement-javas-hashcode-correctly/) and [*equals*](https://www.sitepoint.com/implement-javas-equals-method-correctly/) methods of the objects in the Stream.

{% highlight java %}
Arrays.asList(1, 4, 2, 1, 1, 2, 6).stream().distinct().forEach(System.out::println);
//prints out:
//1
//4
//2
//6
{% endhighlight %}

Although the stream contains multiple ones and twos, only unique integers are printed out.

### Limit

A stream can be truncated with *limit*. This is handy when you don't want to use the whole stream but only a section of it.
Limit accepts an integer specifying the maximum size of the returned stream. If the stream is created from an ordered collection (e.g. List) then the returned stream contains the first n elements of the initial stream. Limit also works if the stream is created from a collection where the order is not defined (e.g. Set). In that case don't assume any order in the stream returned by *limit*.

{% highlight java %}
Arrays.asList(1, 2, 3, 4, 5, 6).stream().limit(3).forEach(System.out::println);
//prints out:
//1
//2
//3
{% endhighlight %}

truncating, limit()
skipping, skip()
finding
max min

## Pipelining

simple diagram of stream pipelining

## Specialized Streams

numeric streams, specialized streams (intStream)

## Parallel Streams
an associative, non-interfering, stateless function
ordering, encounter order
limit is expensive

## Other libraries

Guava, apache, lambdaj

## Other

traversable only once, example
lazy

## Summary
