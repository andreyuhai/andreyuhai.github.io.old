---
layout: post
title: "Design patterns: Observer pattern implementation in Java"
excerpt: Observer pattern is a design pattern with an intent to define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.
modified: 2016-06-29 14:36:56 +0300
categories: articles
tags: [design pattern, java, observer, publish-subscribe, loose coupling]
image:
  feature: 2016-06-21-design-patterns-observer-pattern-implementation-in-java/cover.jpg
  credit: suwalls.com
  creditlink: https://suwalls.com/vector/who-observes-the-observer-28727/
comments: true
share: true
published: true
---

A software design pattern is a solution to a commonly occurring problem. It is not a ready made piece of code or a recipe but rather think of it as a best practice. Implementation details can vary from language to language so don't be alarmed when you see an implementation which differs from the one you're used to.

**Observer pattern**, described in [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612 "Design Patterns: Elements of Reusable Object-Oriented Software"), is a design pattern with an intent to define a one-to-many dependency between objects so that **when one object changes state, all its dependents are notified and updated automatically**.

It is easier to think of it in terms of publishing and subscribing. Subscribers (also known as observers) subscribe to a publisher (also known as a subject or an observable) such that when the state of the publisher changes, it notifies all of its subscribers.

In this post I'm going to go over how to implement this classic pattern in Java.

## Example domain

Do you want to know how many people are in space right now? There's an app for that at [howmanypeopleareinspacerightnow.com](http://www.howmanypeopleareinspacerightnow.com/ "howmanypeopleareinspacerightnow.com"). Let's use that as our example. The website *publishes* how many people are in space and different apps can *subscribe* to get notified when the number changes.

## Class diagram

The following is a class diagram of the proposed solution.

![Observer pattern class diagram]({{ site.url }}/images/2016-06-21-design-patterns-observer-pattern-implementation-in-java/observer_pattern.png "Observer pattern class diagram")

## Create interfaces

Firstly, we need to create interfaces for `Subject` and `Observer`.

{% highlight java %}
public interface Subject {
    void register(Observer observer);
    void remove(Observer observer);
    void notifyObservers();
    int getState();
}
{% endhighlight %}

{% highlight java %}
public interface Observer {
    void update();
}
{% endhighlight %}

## Implement interfaces

Now that we have created the interfaces we need, let's implement them. I'm calling the concrete implementation of the *Subject* interface *DataSource* since it is the source of the original data.

{% highlight java %}
import java.util.ArrayList;
import java.util.List;

public class DataSource implements Subject {

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
        observers.forEach(Observer::update);
    }

    @Override
    public int getState() {
        return peopleInSpace;
    }

    public void setPeopleInSpace(int n) {
        peopleInSpace = n;
        notifyObservers();
    }
}
{% endhighlight %}

This class knows how many people are in space and has a list of all the observers it needs to notify when this number changes. `register()` and `remove()` methods can be used to add or remove observers. A [method reference]({{ site.url }}/articles/four-types-of-method-references-in-java-8/ "Four types of method references in Java 8") is used in `notifyObservers()` to update registered observers. Whenever `peopleInSpace` is changed by `setPeopleInSpace()`, `notifyObservers()` is called to notify all registered listeners about the change of state.

Next, let's have a look at the implementation of the Observer interface. In this example the observer is a hallway display which shows the number of people in space.

{% highlight java %}
public class HallwayDisplay implements Observer {

    private int peopleInSpace;
    private Subject subject;

    public HallwayDisplay(Subject subject) {
        this.subject = subject;
    }

    @Override
    public void update() {
        peopleInSpace = subject.getState();
        display();
    }

    public void display() {
        System.out.println("People in space right now: " + peopleInSpace);
    }
}
{% endhighlight %}

The observer knows its subject and can get the current state of the subject.

## Running the example

A quick way to see if everything is working is to create a new *DataSource* object and register a new *HallwayDisplay* as its observer. When updating the number of people in space, HallwayDisplay should print out the updated data.

{% highlight java %}
DataSource source = new DataSource();
source.register(new HallwayDisplay(source));
source.setPeopleInSpace(5);

//prints out: People in space right now: 5
{% endhighlight %}

## Loose coupling

When two objects know very little of each other but are able to interact, they are considered **loosely coupled**. The observer pattern is a good example of loose coupling. The only thing the subject knows about observers is that they implement a certain interface. This is good because changes to either the subject or the observer will not affect the other. It is possible to add new types of observers at any time while never making any changes to the subject.

## Final words

This is not the end. In the [next post]({{ site.url }}/articles/design-patterns-specialization-of-the-classic-observer-pattern/ "Specialization of the observer pattern") I'm going to look at a specialization of this classic pattern and show how to take advantage of [lambda expressions]({{ site.url}}/articles/java-8-lambda-expressions/ "Java 8 Lambda expressions").
