---
layout: post
title: "Using environment variables with Spring Boot"
excerpt: This post will cover how you can use OS environment variables to pass configuration values to a Spring Boot application.
modified: 2017-12-17 07:17:09 +0200
categories: articles
tags: [spring, spring boot, java]
image:
  feature: 2017-12-17-using-environment-variables-with-spring-boot/cover.jpg
  credit: Gustavo Quepón
  creditlink: https://unsplash.com/photos/pF_2lrjWiJE
comments: true
share: true
published: true
aging: true
---

Spring Boot applications [can be easily configured via property files]({{site.url}}/articles/externalized-configuration-in-spring-boot/ "Externalized configuration in Spring Boot").
But sometimes you need more flexibility in terms of how the configuration is provided to the application.
A popular approach next to configuration files is to use environment variables.
This post will cover how you can use OS environment variables to pass configuration values to a Spring Boot application.

## Injecting environment variables

You might already be familiar with the `@Value` annotation.
It can be used to inject property values to beans.
In addition to looking values from property files, the `@Value` annotation also looks for matching values from the environment.
So when you have an environment property called `DB_URL`, you can inject it to a field with `@Value("${DB_URL}")`.

{% highlight java %}
@Value("${DB_URL}")
private String dbUrl;
{% endhighlight %}

## Setting `application.properties` values from environment

Similar to the previous approach, it is possible to assign values to properties in your `application.properties` file from the environment.

{% highlight yaml %}
api.key=${API_KEY}
{% endhighlight %}

As with the `@Value` annotation, you can provide a default value which will be used if the environment variable is not found.

{% highlight yaml %}
api.key=${API_KEY:123abc}
{% endhighlight %}

## SPRING_APPLICATION_JSON

At application startup, Spring Boot will look for an environment variable called `SPRING_APPLICATION_JSON`.
It can be used to provide a set of application properties using inline JSON.
For example, you can set `ec2.public.url` property as follows.

{% highlight bash  %}
SPRING_APPLICATION_JSON='{"ec2":{"public":{"url":"http://mydomain.com"}}}'
{% endhighlight %}

To get a more thorough overview of all the ways Spring Boot allows you to configure applications, you should definitely look at [the official docs](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html).
You can find an example Sprint Boot application from [Github](https://github.com/indrekots/spring-boot-envvariables) that demonstrates the approaches covered in this post.
