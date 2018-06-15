---
layout: post
title: "Java 8: CompletableFuture"
excerpt: CompletableFuture is an implementation of the Future interface but with a modern twist. It takes advantage of functional operations to promote asynchronous/event-driven programming model.
modified: 2016-04-04 07:20:35 +0300
categories: articles
tags: [java, java 8, multithreading, i/0]
image:
  feature: 2016-04-04-java-8-completablefuture/cover.jpg
  credit: http://thebraincandypodcast.com/
  creditlink: http://thebraincandypodcast.com/episode-3-the-future/
comments: true
share: true
published: true
---

Since the beginning of multicore processors, the most effective way to increase the performance of an application is to fully utilize all processor cores. Running tasks which might take longer to return asynchronously (in a separate thread) is beneficial because **the main thread can continue to do useful work while your result is being calculated**. Using blocking calls can considerably decrease throughput as the program is wasting useful clock cycles on waiting for calls to return. Achieving greater throughput is also possible by [splitting a long running task into smaller tasks and run them in parallel]({{ site.url }}/articles/the-fork-slash-join-framework-in-java/ "The fork/join framework in Java").

CompletableFuture is an implementation of the Future interface but with a modern twist. It takes advantage of **functional operations to promote asynchronous/event-driven programming model**. You should have some knowledge of threads in Java to be able to follow along.

## Future

To better understand CompletableFutures, let's have a look at Futures first. They model asynchronous computations and provide a reference to its result which might not be available immediately. Methods are provided to get the result or check if the computation has finished. Calls to methods returning Futures can return immediately and **the calling thread can continue to do useful work**. All in all, Futures are friendlier to use than lower-level threads.

To create a future you need to wrap an operation in a Callable and pass it to an ExecutorService.

{% highlight java %}
ExecutorService executorService = Executors.newSingleThreadExecutor();

Future<String> result = executorService.submit(() -> {
    delay(5000); //simulate network call
    return "{\"result\":\"success\"}";
});

IntStream.range(1, 10).forEach(i -> {
    delay(100);
    System.out.println("Doing useful work");
});
System.out.println(result.get());
{% endhighlight %}

In the previous example a Callable is passed to an ExecutorService. Since Java 8, instead of an anonymous class you can use a [lambda expression]({{ site.url }}/articles/java-8-lambda-expressions/ "Java 8: Lambda Expressions") to define the body of the Callable. The `delay` method is used to simulate a network call which delays the thread for 5 seconds. ExecutorService will return a Future which can be used later to retrieve the result. During the time the network call is processed, it is possible to do other useful work. Finally calling `get()` on the Future will get its result. If it's not available yet, **the execution is blocked until the result becomes available**. But what if the thread never finishes or takes too long to compute? The [*get* method](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html "Future JavaDoc") can accept a timeout as an argument. You can also use the `isDone()` method to check if the task has completed.

But what if you'd like to take the result of an asynchronous task and immediately do other computations with it and then, say, combine the result with a result from another async task. This is where CompletableFutures come in handy because they allow to express dependencies between tasks declaratively.

## CompletableFuture

The name CompletableFuture comes from the fact that it is a Future which can be explicitly completed by calling the `complete()` method. Instead of using an ExecutorService to return a Future, completing a CompletableFuture can be done explicitly in a thread.

As in the previous example with a Future, let's say we're making a network call to retrieve some data.

{% highlight java %}
public Future<String> getResultOverNetwork() {
    CompletableFuture<String> future = new CompletableFuture<>();
    new Thread(() -> {
        delay(5000); //simulate network call
        future.complete("{\"result\":\"success\"}");
    }).start();
    return future;
}
{% endhighlight %}

The computation (in this case a simulated network call) is done in a separate thread. A [lambda expression]({{ site.url }}/articles/java-8-lambda-expressions/ "Java 8: Lambda Expressions") is used to define the Runnable passed to the thread. Before returning a Future, the thread is started. The `getResultOverNetwork()` method returns almost instantaneously, leaving the calling thread free to to other useful work.

{% highlight java %}
Future<String> future = getResultOverNetwork();
IntStream.range(1, 10).forEach(i -> {
    delay(100);
    System.out.println("Doing useful work");
});
System.out.println(future.get());
{% endhighlight %}

Technically this example is almost the same as shown previously with a regular Future. The only difference is that instead of a plain old Thread, an ExecutorService was used. To be fair though, CompletableFuture class has some **static methods which make the usage a little bit easier** as can be seen in the following example.

{% highlight java %}
public Future<String> getResultOverNetwork() {
    return CompletableFuture.supplyAsync(() -> {
        delay(5000);
        return "{\"result\":\"success\"}";
    });
}
{% endhighlight %}

But still, what's so special about CompletableFutures?

### Pipelining tasks

CompletableFuture allows to compose multiple synchronous and/or asynchronous operations into a pipeline. So instead of calling `get()` and blocking until a Future returns a value, **it is possible to declaratively tell how to react to a value** when it becomes available.

{% highlight java %}
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 4 + 3)
                .thenApply(i -> i*2);
System.out.println(future.get());
//prints out 14
{% endhighlight %}

Although the example is trivial, it shows that the returned value from the first task is pipelined to the next. For example, in a real world scenario this could be used when you need to retrieve some data using a RESTful API and process it immediately afterwards.

In addition to `thenApply()`, the API provides other useful methods. `thenCombine()` can be used to combine the results from 2 tasks and `thenAccept()` consumes the result from a task.

{% highlight java %}
//somewhere in the class header
import static java.util.concurrent.CompletableFuture.supplyAsync;

CompletableFuture<Void> future = supplyAsync(itemService::getFruits).
        thenCombine(supplyAsync(itemService::getVeggies),
        (fruits, veggies) -> Stream.concat(Arrays.stream(fruits), Arrays.stream(veggies))
).thenAccept(items -> items.forEach(System.out::println));
future.join();
{% endhighlight %}

`getFruits()` and `getVeggies()` are implemented as follows.

{% highlight java %}
public String[] getFruits() {
    delay(1500); //simulate network latency
    return new String[]{"apple", "apricot", "banana"};
}

public String[] getVeggies() {
    delay(2000); //simulate network latency
    return new String[]{"broccoli", "brussels sprout"};
}
{% endhighlight %}

In this example, 2 CompletableFutures are created using the `supplyAsync()` static method. They're pipelined using `thenCombine()` which accepts a lambda expression as a second argument describing how to combine the results of the 2 tasks. These tasks are run in parallel and as soon as both become available their results are combined and then the end result is consumed using `thenAccept()` (in this case consumption is done by printing out the members of the array). Method references `itemService::getFruits` and `itemService::getVeggies` simulate network calls and they return an array of strings.

The returned Future is of type `Void` because `thenAccept()` consumes the array. Therefore there's no need to call `get()` (although it is possible) and calling `join()` will suffice.

> `join()` - Returns the result value when complete, or throws an (unchecked) exception if completed exceptionally.

### Async versions

Pipelining methods come with asynchronous versions as well. Meaning that the next step in the pipeline will be executed in a new thread. For example, the API provides `thenAcceptAsync()` which is an asynchronous version of `thenAccept()`.

## Summary

This is not by any means a [full overview of the CompletableFuture API](http://www.nurkiewicz.com/2013/05/java-8-definitive-guide-to.html "thorough overview of the CompletableFuture API"). For example, I did not cover exception handling. Be sure to have a look at the [JavaDoc](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html "CompletableFuture JavaDoc") for all the possibilities or dive into a [book](https://www.manning.com/books/java-8-in-action "Java 8 in Action") for more detailed and thorough explanation.
