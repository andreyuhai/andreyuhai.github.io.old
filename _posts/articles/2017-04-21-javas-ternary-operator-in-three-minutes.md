---
layout: post
title: "Java’s Ternary Operator in Three Minutes"
excerpt: The ternary operator is a form of syntactic sugar for if-then-else statements. It is also known as the conditional operator, which is perhaps a more meaningful name because it evaluates conditions like if does.
modified: 2017-04-21 08:29:28 +0300
categories: articles
tags: [java, conditional, control flow]
image:
  feature: 2017-04-21-javas-ternary-operator-in-three-minutes/cover.jpg
  credit: Scott Webb
  creditlink: https://unsplash.com/@scottwebb?photo=YBwPrBiccX4
comments: true
share: true
published: true
aging: false
canonical-url: https://www.sitepoint.com/java-ternary-operator/
---
*This article was originally published on [SitePoint](https://www.sitepoint.com/java-ternary-operator/) on March 22, 2017. For more interesting content about Java, check out [SitePoint's Java channel](https://www.sitepoint.com/java/ "Java channel at SitePoint").*

The ternary operator is a form of [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar "Syntactic Sugar") for `if`-then-`else` statements.
It is also known as the conditional operator, which is perhaps a more meaningful name because it evaluates conditions like `if` does.
Provided that the operator is used with care, it can make code more concise without sacrificing readability.

This article requires you to have a solid understanding of how [`if`-statements work in Java](https://www.sitepoint.com/javas-if-statement-tutorial/ "Java’s If Statement in Five Minutes").

## Ternary Operator

The _ternary operator_ evaluates a condition and chooses one of two branches to execute.
Like in many other [programming languages](https://en.wikipedia.org/wiki/%3F: "Ternary Operators - ?:"), `?` and `:` symbols are used to form it.
Other common names you might encounter that refer to the same concept are _inline if_, _ternary if_ and the _conditional operator_.

### Syntax

The name *ternary* refers to the fact that the operator takes three operands.

{% highlight java %}
condition ? exprTrue : exprFalse
{% endhighlight %}

The `condition` is a boolean expression that evaluates to either `true` or `false`.
Both, `exprTrue` and `exprFalse` are also expressions but they can evaluate to anything you want them to ([except `void`](https://docs.oracle.com/javase/specs/jls/se7/html/jls-15.html#jls-15.25)).
If the condition is `true`, the ternary operator evaluates `exprTrue`.
Otherwise `exprFalse` is evaluated.

The ternary operator is an expression (like `price + 20` for example), which means that once executed, it has a value.
And that value needs to be further used, for example by assigning it to a variable or returning from a method, or the expression will not compile.

It's worth mentioning that the operator is [lazy](https://www.sitepoint.com/java-in-praise-of-laziness/) in the sense that only the used expression is evaluated:
The ternary operator will not evaluate the unused branch.

## Examples

As you can see, the basic structure is very similar to an `if`-then-`else` statement but it is condensed to a single line.
Let's have a look at a concrete example.
It demonstrates how a simple `if`-then-`else` statement can be replaced with a ternary operator.

{% highlight java %}
int price;
if (isPremiumMember()) {
    price = 80;
}
else {
    price = 100;
}

// is equivalent to

int price = isPremiumMember() ? 80 : 100;
{% endhighlight %}

The `price` variable gets a value based on whether the user is a premium member or not.
As you can see, the ternary operator is succinct and in this case improves readability.

Since you can use it as an expression, oftentimes it enables you to remove multiple return statements in a method by replacing them with a single expression.

{% highlight java %}
int price() {
    if (isPremiumMember()) {
        return 80;
    }
    else {
        return 100;
    }
}
{% endhighlight %}

Can be replaced with

{% highlight java %}
int price() {
    return isPremiumMember() ? 80 : 100;
}
{% endhighlight %}

The refactored `price` method is considerably shorter and as legible as before.

### Nesting Multiple Operators

Java allows to nest one conditional operator into another one.
Be careful when doing that though.
Nested conditional operators can hinder readability.

{% highlight java %}
int amount = 10;
String result = amount <= 2 ? "couple" : amount > 2 && amount < 5 ? "few" : amount <= 5 ? "several" : "many";
{% endhighlight %}

Although the example can be improved with formatting, sometimes it's better to avoid using the ternary operator and go with the plain old [`if`-then-`else`](https://www.sitepoint.com/javas-if-statement-tutorial/ "Java’s If Statement in Five Minutes") or [switch statement](https://www.sitepoint.com/javas-switch-statement/ "Java’s Switch Statement in Three Minutes") route.

## Summary

You've learned that the ternary operator allows you to shorten an `if`-then-`else` statement to a single line `condition ? exprTrue : exprFalse`.
If done wisely, it makes the code more concise and improves readability.

[Fewer lines of code is not always better, though](http://softwareengineering.stackexchange.com/a/203686).
It's easy to overuse the ternary operator and [make your code less readable](http://softwareengineering.stackexchange.com/a/28315).
Use common sense and keep in mind that [programs must be written for people to read, and only incidentally for machines to execute](https://www.goodreads.com/quotes/9168-programs-must-be-written-for-people-to-read-and-only "Harold Abelson, Structure and Interpretation of Computer Programs").
A more readable way to express conditions, particularly if there are many of them, can be the [switch statement](https://www.sitepoint.com/javas-switch-statement/ "Java’s Switch Statement in Three Minutes").
