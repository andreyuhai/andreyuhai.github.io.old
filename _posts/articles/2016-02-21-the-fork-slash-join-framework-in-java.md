---
layout: post
title: "The fork/join framework in Java"
excerpt: Introduced in Java 7, the fork/join framework is used to divide a parallelizable task into smaller ones and afterwards combine the intermediate results to get the end result.
modified: 2016-02-21 19:45:15 +0200
categories: articles
tags: [java, fork, join, concurrency, concurrent programming, fork/join]
image:
  feature: 2016-02-15-the-fork-slash-join-framework-in-java/cover.jpg
  credit: John D. Williams
  creditlink:
comments: true
share: true
published: true
---

Introduced in Java 7, the fork/join framework is used to divide a parallelizable task into smaller ones and afterwards combine the intermediate results to get the end result. It is an implementation of the `ExecutorService` interface. In this post we're going to look at how to use the fork/join framework. We will create subtasks and submit them to the thread pool.

## Getting started

In essence, you need to create a task by extending the `RecursiveTask<R>` abstract class and implementing the `compute()` method. The contents of the `compute()` method can be expressed with the following pseudocode.

{% highlight java %}
if (task small enough) {
  do work sequentially
}
else {
  divide the task
  call the tasks recursively
}
{% endhighlight %}
The implementation of [RecursiveTask](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/RecursiveTask.html "RecursiveTask documentation") should be passed to an instance of [ForkJoinPool](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html "ForkJoinPool documentation"). If you're familiar with the concept of **divide and conquer** then you can see that fork/join is a parallel version of that. I hope that the following diagram will help illustrate it.

![fork/join diagram]({{ site.url }}/images/2016-02-15-the-fork-slash-join-framework-in-java/fork_join_diagram.png "fork/join diagram")

## What if I don't want to return a result

*RecursiveTask* is a recursive result-bearing *ForkJoinTask*. If you only want to modify an existing data structure, what should be the return value of *RecursiveTask*? When your task returns no result, you should extend the [RecursiveAction](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/RecursiveAction.html "RecursiveAction documentation") abstract class which is a recursive resultless *ForkJoinTask*.

## Example code

Most of the time it's easier to just look at the code. Let's implement a very simple (and mostly useless) use case of the fork/join framework in Java. Summing the elements of an array should be a good place to start. First of all the following is an implementation of a *RecursiveTask*.

{% highlight java %}
public class LongSum extends RecursiveTask<Long> {

    private static final int LIMIT = 1000;

    private long[] numbers;
    private int start;
    private int end;

    public LongSum(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;
        if (length <= LIMIT) {
            return computeSequentially();
        }

        LongSum left = new LongSum(numbers, start, length/2);
        left.fork();

        LongSum right = new LongSum(numbers, start + length/2, end);
        Long rightResult = right.compute();
        Long leftResult = left.join();

        return leftResult + rightResult;
    }

    private Long computeSequentially() {
        return Arrays.stream(numbers).sum();
    }
}
{% endhighlight %}

The `compute()` method first checks if the size of the task is small enough. If it is, then the result is computed sequentially. Otherwise the task is split into two. `fork()` is called on the first half, meaning that it is **going to be executed asynchronously** in a separate thread. The result of the other half is computed in the current thread. Eventually `join()` is called on the first half. This blocks the current thread until a result is returned. **Keep in mind that you need to call `join()` after the computation of both subtasks has started**. Otherwise you'll end up with a sequential algorithm and that defeats the purpose of using the fork/join framework.

To use the implementation of *RecursiveTask* you need to instantiate it and pass it to a *ForkJoinPool*.

{% highlight java %}
long[] numbers = LongStream.rangeClosed(1, 10000).toArray();
LongSum longSum = new LongSum(numbers, 0, numbers.length);
Long result = new ForkJoinPool().invoke(longSum);
{% endhighlight %}

## Conclusion

Concurrent programming can be difficult. If you're faced with a task that fits within the domain of the fork/join framework then you're in luck. Coming up with a multi-threaded algorithm yourself is error prone unless you're an expert in the field. When the fork/join framework does not suit your needs, you need to use other concurrency constructs provided by Java. I would definitely advise reading [Java Concurrency in Practice](http://jcip.net/ "Java Concurrency in Practice homepage") if you're interested learning more about concurrent programming in Java.
