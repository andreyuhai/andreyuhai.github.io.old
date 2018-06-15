---
layout: post
title: "Creating a custom collector in Java 8"
excerpt: A collector is a recipe for how to build a summary of the elements in a Stream. In this post you'll see how to create a custom collector in Java 8.
modified: 2018-03-11 11:32:07 +0200
categories: articles
tags: [java, java 8, stream, collector, collect, reduce, fold]
image:
  feature: 2015-12-28-creating-a-collector-in-java-8/cover.jpg
  credit:
  creditlink:
comments: true
share: true
published: true
---

[Streams]({{site.url}}/articles/5-ways-to-create-a-stream-in-java-8/ "5 ways to create a Stream in Java 8") help you process collections in a declarative manner. They support two types of operations: intermediate and terminal. While intermediate operations convert a stream to another stream, terminal operations consume the stream and return the final result. Java 8 [Stream interface](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html "Java 8 API Stream interface") defines a `collect` method which performs a mutable reduction operation on the elements of the stream. It accepts a `Collector` as a parameter which encapsulates the strategy that is used to compute the final result. In this post we'll have a look at how to create a custom collector in Java 8 from scratch.

## What is a collector?

A collector is a recipe for how to build a summary of the elements in a Stream. If you're familiar with Streams then you have probably seen the `toList()` collector.

{% highlight java %}
books.stream().
  filter(b -> b.getPageCount() > 100).
  collect(toList());
{% endhighlight %}

This collects all elements in a stream into a list. Among others, the `toList()` collector is part of the `Collectors` class which includes several other collectors as well.

## Existing collectors

To understand what collectors do and how they work, let's have a look at existing collectors in the Java API. In general, collectors can be divided into three broader categories:

* Reducing and summarizing stream elements to a single value
* Grouping stream elements
* Partitioning stream elements

## Collector returning a single value

An example of a collector which returns a single value is `counting()`. This counts the number of elements in a stream. Other notable collectors in this group include: `maxBy()`, `minBy()`, `summingInt()`.

### Collector which groups elements

A common database query might include a *group by* statement. It is possible to implement it with Java with an imperative style but it is cumbersome and very verbose. A `groupingBy()` collector can be used to ease the pain of verbosity. The following is an example of a grouping collector.

{% highlight java %}
Map<String, List<Book>> booksByAuthor = books.stream().
  collect(groupingBy(Book::getAuthor));  
{% endhighlight %}

### Partitioning collector

Partitioning is technically a special case of grouping. A predicate (function which returns a boolean) is used to divide the stream into two groups. It returns a *Map* and its keys are booleans. So for example, to partition a stream of books into long and short ones you can use the following expression:

{% highlight java %}
Map<Boolean, List<Book>> booksByLength = books.stream().
  collect(Collectors.partitioningBy(b -> b.getPageCount() > 500));
{% endhighlight %}

## Collector interface

The [Collector interface](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.html "Collector interface javadoc page") defines a set of methods which are used during the reduction process. The following is the interface signature with the five methods it declares.

{% highlight java %}
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
}
{% endhighlight %}

* T is the type of items in the stream to be collected
* A is the type of the accumulator
* R is the type of the result returned by the collector

### Supplier

The `supplier()` must return a function that creates an empty accumulator. This will also represent the result of the collection process when applied on an empty stream.

### Accumulator

The job of the `accumulator()` is to return a function which performs the reduction operation. It accepts two arguments. First one being the mutable result container (accumulator) and the second one the stream element that should be folded into the result container.

### Finisher

The `finisher()` returns a function which performs the final transformation from the intermediate result container to the final result of type R. Often times the accumulator already represents the final result, so the finisher can return the *identity* function.

### Combiner

When the stream is collected in parallel then the `combiner()` method is used to return a function which knows how to merge two accumulators.

### Characteristics

Finally, the `characteristics()` method returns an immutable set of `Characteristics` which define the behavior of the collector. This is used to check which kind of optimizations can be done during the reduction process. For example, if the set contains `CONCURRENT`, then the collection process can be performed in parallel.

## Building a custom collector

In the previous paragraph a general overview of the *Collector* interface was given. This should be enough to start creating our own custom collector. The Collectors class includes static methods which return some commonly used Collectors. But for special cases, we would need to create our own custom collector. Suppose you have a list of continuous values and you would like to create a histogram from it. A histogram is a graphical representation of the distribution of numeric data. The custom collector would need to return a data structure which holds all the required data to create a histogram.

Since the values are continuous, they need do be divided into buckets. To make things simpler, let's create a collector which can be applied to a stream of `double` values and which returns a `Map<Integer, Integer>`. The key of the map is the bucket index and the value counts the number of occurrences of values from the original stream that fit in the bucket bounds.

To create a new collector, a new class needs to be created which implements the Collector interface. Let's call the class `HistogramCollector`.

{% highlight java %}
public class HistogramCollector
    implements Collector<Double, Map<Integer, Integer>, Map<Integer, Integer>> {

    private int bucketSize;

    public HistogramCollector(int bucketSize) {
        this.bucketSize = bucketSize;
    }

    //TODO: implement abstract methods defined in the Collector interface
}
{% endhighlight %}

Its constructor accepts the size of the bucket. So for example, if the size is set to 10, values from 0 to 10 (excluding 10) will be in the 0th bucket.

### Implementing the interface

The methods defined by the interface need to be implemented. I'm going to implement them in the order defined in the previous paragraph. First of all, the `supplier()` method needs to return a function which returns an empty accumulator.

{% highlight java %}
@Override
public Supplier<Map<Integer, Integer>> supplier() {
    return HashMap::new;
}
{% endhighlight %}

When the stream is being traversed, stream elements are being accumulated by the function returned by the `accumulator()` method.

{% highlight java %}
@Override
public BiConsumer<Map<Integer, Integer>, Double> accumulator() {
    return (map, val) -> map.merge((int)(val / bucketSize), 1,
            (a, b) -> a + 1);
}
{% endhighlight %}

The returned function accepts the accumulator map and the next element in the stream. Then it merges the element into the map. The map key is found by dividing the value by the size of the bucket.

The `finisher()` method needs to return a function which transforms the accumulator to the final result. In this case, the accumulator is the final result as well. Therefore it is possible to return the identity function.

>In mathematics, an identity function, also called an identity relation or identity map or identity transformation, is a function that always returns the same value that was used as its argument -- Wikipedia

{% highlight java %}
@Override
public Function<Map<Integer, Integer>, Map<Integer, Integer>> finisher() {
    return Function.identity();
}
{% endhighlight %}

Two more methods need to be implemented. The `combiner()` method is used when 2 maps are merged. This can happen when you process the stream in parallel. When merging maps, the values for the same keys should be summed.

{% highlight java %}
@Override
public BinaryOperator<Map<Integer, Integer>> combiner() {
    return (map1, map2) -> {
        map2.forEach((k, v) -> map1.merge(k, v, (v1, v2) -> v1 + v2));
        return map1;
    };
}
{% endhighlight %}

The final method is `characteristics()`. This returns a Set of `Characteristics` indicating the characteristics of this Collector, which are used by the collection process. In our case we can return `IDENTITY_FINISH` and `UNORDERED`.

| Characteristic  | Description  |
|:--|:--|
| IDENTITY_FINISH  | Indicates that the `finisher()` function is the identity function and can be left out  |
| UNORDERED  | Indicates that the collection operation does not commit to preserving the encounter order of input elements.  |

### Final touch

The *Collectors* class contains static methods for commonly used collectors. Let's create a static method in the *HistogramCollector* class as well. It should returns a new `HistogramCollector`.

{% highlight java %}
public static HistogramCollector toHistogram(int bucketSize) {
  return new HistogramCollector(bucketSize);
}
{% endhighlight %}

### Collector in action

When all methods defined by the *Collector* interface are implemented, then let's see the collector in action.

{% highlight java %}
@Test
public void histogramCollectTest() throws Exception {
    List<Double> numbers = Arrays.asList(1.0, 1.1, 1.4, 1.7, 1.4, 5.4, 9.9);
    Map<Integer, Integer> histogram = numbers.stream().collect(toHistogram(1));

    ImmutableMap<Integer, Integer> expected = ImmutableMap.<Integer, Integer>builder().
            put(1, 5).put(5, 1).put(9, 1).build();

    assertEquals(expected, histogram);
}
{% endhighlight %}

## Collect vs Reduce

If you're somewhat familiar with Streams, then you'll probably wonder why not use the `reduce()` method. Most of the time you can achieve the same result. There's a semantic difference. The `reduce()` method should combine two values and return a new one, meaning that the reduction (folding) process should be immutable. Whereas the `collect()` method is designed to mutate a container to accumulate the result it’s supposed to produce.
