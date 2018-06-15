---
layout: post
title: "Design patterns: Specialization of the classic observer pattern"
excerpt: In the typical case, the observer is responsible of getting the updated state from the subject. This means that the observer has to hold a reference to the subject (also the subject holds a reference to the observer, meaning we have created a circular reference).
modified: 2016-07-04 11:19:40 +0300
categories: articles
tags: [design pattern, java, java 8, observer, lambda]
image:
  feature: 2016-06-21-design-patterns-observer-pattern-implementation-in-java/cover.jpg
  credit: suwalls.com
  creditlink: https://suwalls.com/vector/who-observes-the-observer-28727/
comments: true
share: true
published: true
---

[Observer pattern]({{ site.url }}/articles/design-patterns-observer-pattern-implementation-in-java/ "Observer pattern implementation in Java") is a classic design pattern with an intent to define a one-to-many dependency between objects so that **when one object changes state, all its dependents are notified and updated automatically**. In the [previous post]({{ site.url }}/articles/design-patterns-observer-pattern-implementation-in-java/ "Observer pattern implementation in Java") I implemented the pattern the classical way, where the observer is responsible of getting the state from the subject. In this post I'll show a specialization of this pattern. This is a continuation of the [previous post]({{ site.url }}/articles/design-patterns-observer-pattern-implementation-in-java/ "Observer pattern implementation in Java"). So if you have not already had a look at it, I humbly advise you to do so. Otherwise some concepts might not be clear to you.

## Providing the state as a parameter

In the typical case, the observer is responsible of getting the updated state from the subject. This means that the observer has to hold a reference to the subject (also the subject holds a reference to the observer, meaning we have created a circular reference). **All the observer cares about is the updated state, not the subject itself**. A specialization of this pattern dictates that the updated state should be passed as a parameter to the `update()` method on the observer.

## Code examples

I'm going to use the same example domain as in the [previous post]({{ site.url }}/articles/design-patterns-observer-pattern-implementation-in-java/ "Observer pattern implementation in Java"). In short, we have a subject which publishes to its subscribers [how many people are in space right now](http://www.howmanypeopleareinspacerightnow.com/). Most of the code will remain the same but let's look at the differences. First of all, the interfaces have changed slightly.

{% highlight java %}
public interface Observer {
    void update(int state);
}

public interface Subject {
    void register(Observer observer);
    void remove(Observer observer);
    void notifyObservers();
}
{% endhighlight %}

The `update()` method takes the state as a parameter. Additionally `getState()` method is removed from the *Subject* interface.

{% highlight java %}
public class DataSource implements Subject{

    private int peopleInSpace;
    private List<Observer> observers;

    public DataSource() {
        observers = new ArrayList<>();
    }

    @Override
    public void register(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void remove(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        observers.forEach(o -> o.update(peopleInSpace));
    }

    public void setPeopleInSpace(int n) {
        peopleInSpace = n;
        notifyObservers();
    }
}
{% endhighlight %}

The only noticeable difference (other than the removal of `getState()`) in the concrete implementation of the *Subject* interface is `notifyObservers()`. The updated state is passed as an argument to the `update()` method.

As you might imagine, the only change in the concrete implementation of the *Observer* interface is the `update()` method.

{% highlight java %}
public class HallwayDisplay implements Observer {

    private int peopleInSpace;
    private Subject subject;

    public HallwayDisplay(Subject subject) {
        this.subject = subject;
    }

    @Override
    public void update(int state) {
        peopleInSpace = state;
        display();
    }

    public void display() {
        System.out.println("People in space right now: " + peopleInSpace);
    }
}
{% endhighlight %}

## Creating observers with lambda expressions

The fact that the updated state is passed as a parameter and the observer does not have to hold a reference to the subject, it is easy to create new observers with [lambda expressions]({{ site.url}}/articles/java-8-lambda-expressions/ "Lambda expressions in Java 8").

{% highlight java %}
subject.register(state -> System.out.println("New state is " + state));
{% endhighlight %}

Keep in mind that this will only work if your *Observer* interface defines one and only one method. If you wish, you can also use anonymous inner classes to create new observers.

{% highlight java %}
subject.register(new Observer() {
    @Override
    public void update(int state) {
        System.out.println("New observer: people in space right now: " + state);
    }
});
{% endhighlight %}

## Conclusion

The observer pattern is used a in a lot of frameworks and libraries. Many GUI toolkits use it under the hood. The JDK includes [Observer](https://docs.oracle.com/javase/8/docs/api/java/util/Observer.html) and [Observable](https://docs.oracle.com/javase/8/docs/api/java/util/Observable.html) interfaces so you don't have to implement everything by yourself. Keep your eyes open and you might see this pattern implemented in some shape or form.
