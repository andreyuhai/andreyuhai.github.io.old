---
layout: post
title: "Java’s If Statement in Five Minutes"
excerpt: The if statement is the most basic conditional statement. It checks a condition, which is any boolean expression, and runs a block of code if it is true.
modified: 2017-03-23 20:12:07 +0200
categories: articles
tags: [java, control flow, conditional]
image:
  feature: 2017-03-23-javas-if-statement-in-five-minutes/cover.jpg
  credit: Martin Sanchez
  creditlink: https://unsplash.com/@mzeketv?photo=QhnC70oIVWQ
comments: true
share: true
published: true
aging: false
canonical-url: https://www.sitepoint.com/javas-if-statement-tutorial/
---
*This article was originally published on [SitePoint](https://www.sitepoint.com/javas-if-statement-tutorial/) on March 8, 2017. For more interesting content about Java, check out [SitePoint's Java channel](https://www.sitepoint.com/java/ "Java channel at SitePoint").*

Conditional statements are fundamental for [imperative programming](https://en.wikipedia.org/wiki/Imperative_programming "Imperative programming") languages, including Java. They're used to instruct a program to act differently based on whether something is true or false. Java's `if` statement is the most basic conditional statement - it evaluates a boolean expression and executes code based on its outcome.

To follow along, you need to have a basic understanding of [equality, relational, and conditional](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/opsummary.html "Summary of Operators in Java") operators and how to form [boolean expressions](https://en.wikipedia.org/wiki/Boolean_expression "Boolean expressions") with them. You should be good to go if you get why `1 > 0` evaluates to `true` and `num == 5` evaluates to `false` when `num` equals anything other than `5`.

## The `if` Statement

The `if` statement is the most fundamental control flow statement. Once you understand it, the others will come easily. Essentially, an `if` statement tells a program to execute the following block of code only if the accompanying condition is true.

Here you can see the anatomy of an `if` statement:

{% highlight java %}
int num = 5;

if (num == 5) {
    System.out.println("This message gets printed because num is 5.");
}
{% endhighlight %}

A variable `num` is declared and set to `5`. What comes after that is the `if` statement.

It starts with the keyword `if` followed by a pair of parenthesis. Between the parenthesis you need to provide a _condition_. A condition is a boolean expression - something that evaluates to either `true` or `false`. It can be a variable of type `boolean`, [equality, relational, or conditional](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/opsummary.html "Summary of Operators in Java") expressions (like `num == 5`), or even a method call that returns a `boolean`. [Boolean object wrappers](https://docs.oracle.com/javase/8/docs/api/java/lang/Boolean.html) are also valid.

After the parenthesis you can see a pair of curly braces defining a block of code, often called the _if block_ or _if branch_. That code is only executed **if** the condition evaluated to `true`.

It is common practice to indent your if block as it provides a visual hint for readers. For code blocks that contain only a single line of code you _can_ omit the curly braces - whether you _should_ is [a different discussion](http://stackoverflow.com/q/8020228).

### The `if-else` Statement

In many cases you might want to do two different things based on whether a condition is true or not. This is commonly referred to as `if`-then-`else`. **If** a condition holds true, **then** execute the following block of commands, otherwise (**else**) execute a different block, often called the _else block_ or _else branch_. With this you can optionally provide an alternative execution path that will be followed if the condition evaluates to `false`.

{% highlight java %}
int num = 5;

if (num > 10) {
    // if branch
    System.out.println(
            "This will not be printed, since num is not greater than 10.");
} else {
    // else branch
    System.out.println(
            "This will be printed because the condition is false.");
}
{% endhighlight %}

This example is fairly similar to the previous one - again `num` is declared and set to `5`. But this time the boolean test is slightly different! Only if `num` is greater than `10`, will the expression evaluate to `true`. Since this is not the case, the test evaluates to `false`. According to the rules we defined earlier, the following block of code cannot be executed. Instead, the program's execution jumps directly to the `else` block to handle the `false` case.

### The `if-else-if` Statement

Sometimes you might want to test against multiple boolean expressions, so plain `if`-then-`else` does not quite cut it.
For example, **if** a condition holds true, **then** do something, **else if** another condition holds true, **then** do that another thing. You can of course nest a new `if` statement inside an `else` block but that gets rather unreadable.

Instead it is common practice in Java to pull the second `if` up into the same line as the `else`, thus chaining the statements together. This allows you to check against multiple conditions and pick the first block of code that passes its boolean test.

{% highlight java %}
String callsign = "Maverick";

if ("Iceman".equals(callsign)) {
    System.out.println("You must be Lt. Tom Kazanski");
}
else if ("Maverick".equals(callsign)) {
    System.out.println("You must be Lt. Pete Mitchell");
}
else {
    System.out.println("I'm sorry, I believe we have not met");
}
{% endhighlight %}

In this example, we have a String variable `callsign`. The program first checks if it equals `"Iceman"`. This obviously returns `false`, so the execution jumps into the first `else` where a new `if` waits. Then `callsign` is compared against `"Maverick"`. This returns `true` and the following block of code is executed.

You can make use of the `else` statement to provide a default execution path, which will be executed if none of the conditions evaluated to `true`. In an `if-else-if` chain, only one block of code gets executed, others are ignored.

## Summary

The `if` statement is the most basic conditional statement. It checks a condition, which is any boolean expression, and runs a block of code if it is true. The `else` keyword provides an alternative execution path that gets executed if the condition is false. To test multiple conditions you can chain `if` statements with `else if`.

Although this article gave you an overview of the variations of `if` statements, it did not cover all the conditional statements available in Java. First of all, we did not talk about the [ternary operator](https://www.sitepoint.com/java-ternary-operator/ "Java’s Ternary Operator in Three Minutes"), which can be summarized as a shortcut for the `if` statement. Long `else if` chains can potentially be replaced with [switch statements](https://www.sitepoint.com/javas-switch-statement/ "Java’s Switch Statement in Three Minutes"). And Sometimes it is even possible to avoid `if` statements altogether using [dynamic dispatch](https://en.wikipedia.org/wiki/Dynamic_dispatch "Dynamic dispatch").
