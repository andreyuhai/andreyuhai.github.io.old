---
layout: post
title: "Java's While and Do-While Loops in Five Minutes"
excerpt: A while loop is a control flow statement that runs a piece of code multiple times. It consists of a loop condition and body. Java also has a do while loop.
modified: 2017-05-23 07:34:40 +0300
categories: articles
tags: [java, basics, loop]
image:
  feature: 2017-05-23-javas-while-and-do-while-loops-in-five-minutes/cover.jpg
  credit: Claire Satera
  creditlink: https://unsplash.com/@daisybisley?photo=0lk4hww7pdo
comments: true
share: true
published: true
aging: false
canonical-url: https://www.sitepoint.com/javas-while-and-do-while-loops-tutorial/
---
*This article was originally published on [SitePoint](https://www.sitepoint.com/javas-while-and-do-while-loops-tutorial/) on April 26, 2017. For more interesting content about Java, check out [SitePoint's Java channel](https://www.sitepoint.com/java/ "Java channel at SitePoint").*

A while loop is a [control flow](https://en.wikipedia.org/wiki/Control_flow) statement that allows us to run a piece of code multiple times. Like loops in general, a `while` loop can be used to repeat an action as long as a condition is met. The structure of Java's while loop is very similar to a conditional statement in the sense that they both check a boolean expression and maybe execute some code.

To be able to follow along, this article expects that you understand [variables and arrays](https://www.sitepoint.com/beginning-java-data-types-variables-and-arrays/ "Beginning Java: Data types, Variables, and Arrays") in Java. It would also be good if you had some experience with [conditional expressions](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op2.html "Equality, Relational, and Conditional Operators").

## While Loop

The while loop is the most basic loop construct in Java.
It consists of the `while` keyword, the loop _condition_, and the loop _body_.

{% highlight java %}
while (condition) {
    // loop body
}
{% endhighlight %}

A single run-through of the loop body is referred to as an [_iteration_](https://en.wikipedia.org/wiki/Iteration#Computing "Iteration in computing"). Before each iteration, the loop condition is evaluated and, just like with [if statements](https://www.sitepoint.com/javas-if-statement-tutorial/), the body is executed only if the loop condition evaluates to `true`. In fact, a `while` loop body is repeated as long as the loop condition stays `true` - you can think of them as `if` statements where the body of the statement can be repeated.

Usually some execution of the loop will change something that makes the condition evaluate to `false` and thus the loop ends. A loop with a condition that never becomes `false` runs infinitely and is commonly referred to as an [infinite loop](https://en.wikipedia.org/wiki/Infinite_loop "Infinite loop").

### Example

{% highlight java %}
int num = 0;
while (num < 10) {
    System.out.println(num);
    num++;
}
{% endhighlight %}

This example prints out numbers from 0 to 9. If this seems foreign to you, don't worry. We'll go through it step by step.

We first initialize a variable `num` to equal `0`. This will be our [loop counter](https://en.wikipedia.org/wiki/For_loop#Loop_counter "Loop Counter"). When the program encounters a `while` statement, its condition will be evaluated. In our case `0 < 10` evaluates to `true` and the loop body is executed.
Inside the loop body, the `num` variable is printed out and then incremented by one. Here is where the first iteration ends.

After the first run-through of the loop body, the loop condition is going to be evaluated for the second time. `1 < 10` still evaluates to `true` and the next iteration can commence. As you can imagine, the same process will be repeated several more times.

The final iteration begins when `num` is equal to `9`. Our loop counter is printed out the last time and is incremented to equal `10`. This time, however, a new iteration cannot begin because the loop condition evaluates to `false`. `10` is not smaller than `10`.

As you can see, the loop ran as long as the loop condition held `true`. Armed with this knowledge, you can create `while` loops that are a bit more complex, but on the other hand, more useful as well. Let's iterate over an array.

{% highlight java %}
String[] names = {"Doc", "Dopey", "Bashful", "Grumpy", "Sneezy", "Sleepy", "Happy"};
int index = 0;
while (index < names.length) {
    System.out.println(names[index]);
    index++;
}
{% endhighlight %}

The general concept of this example is the same as in the previous one. We initialize a [loop counter](https://en.wikipedia.org/wiki/For_loop#Loop_counter "Loop counter in for loops") and iterate over an array until all elements in the array have been printed out. As a matter of fact, iterating over arrays (or [Collections](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html "Collection interface") for that matter) is a very common use case and Java provides a loop construct which is better suited for that - [the `for` loop](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/for.html "The for Statement").

## Do-While Loop

A `do-while` loop is very similar to a `while` loop but there is one significant difference: Unlike with a `while` loop, the condition is checked at the _end_ of each iteration. This means that a `do-while` loop is always executed at least once.

{% highlight java %}
do {
    // loop body
}
while (condition);
{% endhighlight %}

### Example

A `do-while` loop first executes the loop body and then evaluates the loop condition. Based on the result of the evaluation, the loop either terminates or a new iteration is started. To illustrate this idea, let's have a look at a simple _guess my name_ game.

{% highlight java %}
Scanner scanner = new Scanner(System.in);
String guess;
do {
    System.out.print("Guess my name: ");
    guess = scanner.nextLine();
}
while (!"Daffy Duck".equals(guess));
System.out.println("Congratulations, you guessed my name correctly!");
{% endhighlight %}

The example uses a `Scanner` to parse input from `System.in`. This is the standard input stream which in most cases corresponds to keyboard input. To put it simply, we're going to read text typed by the player.

In a guessing game we would like to prompt the player for an answer at least once and do it until the player guesses the correct answer. A `do-while` loop fits perfectly here. In the loop body we receive input from the player and then the loop condition checks whether it is the correct answer or not. The loop must run as long as the guess does not [equal](https://www.sitepoint.com/implement-javas-equals-method-correctly/) `Daffy Duck`. If a correct answer is received, the loop terminates and we congratulate the player.

## Summary

Loops allow you to repeat a block of code multiple times. This article covered the `while` and `do-while` loops in Java.
They're relatively similar in that both check a condition and execute the loop body if it evaluated to `true` but they have one major difference: A `while` loop's condition is checked before each iteration - the loop condition for `do-while`, however, is checked at the end of each iteration. This means that a `do-while` loop is always executed at least once.

In addition `while` and `do-while`, Java provides [other loop constructs](https://www.tutorialspoint.com/javaexamples/method_for.htm "For and for-each loops in Java") that were not covered in this article. And if you're interested enough, you can have a look at [recursion](https://en.wikipedia.org/wiki/Recursion_(computer_science)).
Sometimes it's possible to use a recursive function instead of loops.
