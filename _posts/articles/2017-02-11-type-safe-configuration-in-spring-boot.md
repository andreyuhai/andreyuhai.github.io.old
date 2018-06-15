---
layout: post
title: "Type-safe configuration in Spring Boot"
excerpt: Spring Boot provides a type-safe way to handle configuration by allowing you to create a bean and populating it with property values from your configuration file.
modified: 2017-01-14 13:13:48 +0200
categories: articles
tags: [java, spring, spring boot, configuration]
image:
  feature: 2016-12-31-externalized-configuration-in-spring-boot/cover.jpg
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

[Spring Boot makes is very easy to read values from an external properties file]({{site.url}}/articles/externalized-configuration-in-spring-boot/ "Externalized configuration in Spring Boot"). Without having to do any configuration, you are able to inject values to instance fields using the `@Value` annotation or using Spring's `Environment` abstraction. But both of these solutions require you to [specify the property value as a string](https://tuhrig.de/why-using-springs-value-annotation-is-bad/ "Why using Spring’s @Value annotation is bad"). Spring Boot also provides a type-safe way to handle configuration by allowing you to create a bean and populating it with property values from your configuration file.

## Introducing @ConfigurationProperties

Let's have a look at the following example `application.yml` file (this will work with a `.properties` file as well).

{% highlight yaml %}
application:
    emailServer: 192.168.1.2
    serviceUser: user
    servicePassword: password
{% endhighlight %}

This configuration file has three properties nested under the `application` namespace. The following is a Java class declaring these  properties as instance variables.

{% highlight java %}
@ConfigurationProperties("application")
public class ApplicationProperties {

    private String emailServer;
    private String serviceUser;
    private String servicePassword;

    //getters and setters
}
{% endhighlight %}

Observant readers noticed that this class is annotated with `@ConfigurationProperties("application")`. This instructs Spring Boot to bind property values from your configuration file to instance fields. `"application"` specifies that only properties with the given prefix should be bound to this object.

Before you can start injecting `@ConfigurationProperties` beans you need to enable configuration properties by adding `@EnableConfigurationProperties(ApplicationProperties.class)` to a configuration bean.

{% highlight java %}
@SpringBootApplication
@EnableConfigurationProperties(ApplicationProperties.class)
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
{% endhighlight %}

Now that this is done, feel free to inject `ApplicationProperties` as you would inject any other bean.

{% highlight java %}
@RestController
public class TestResource {

    private final ApplicationProperties applicationProperties;

    @Autowired
    public TestResource(ApplicationProperties applicationProperties) {
        this.applicationProperties = applicationProperties;
    }

    @GetMapping
    public String get() {
        return applicationProperties.getServicePassword();
    }
}
{% endhighlight %}

## Configuration validation

Having configuration properties defined in a bean enables you to validate them using [Java Bean Validation](http://beanvalidation.org/1.0/spec/ "JSR 303: Bean Validation"). Let's say the `servicePassword` property must always be defined. Otherwise your application's state is invalid. That can be easily achieved by annotating the field with `@NotNull`.

{% highlight java %}

@NotNull
private String servicePassword;

{% endhighlight %}

If the application is started with an `applicaiton.yml` that does not include `servicePassword`, Spring Boot throws an exception on startup and the application fails to start.

{% highlight yaml %}
***************************
APPLICATION FAILED TO START
***************************

Description:

Binding to target io.indrek.ApplicationProperties@58a55449 failed:

    Property: application.servicePassword
    Value: null
    Reason: may not be null


Action:

Update your application's configuration


Process finished with exit code 130 (interrupted by signal 2: SIGINT)
{% endhighlight %}

## Nested configuration

What if I add another level of nesting into my `applicaiton.yml` file? Can I still use my `ApplicationProperties` as a single source of configuration values? The answer to this question is yes. But you need to do some modifications first. In the following example, the application has been configured with a `remoteHost`.

{% highlight yaml %}
application:
    emailServer: 192.168.1.2
    serviceUser: user
    servicePassword: password
    remoteHost:
        hostname: 192.168.1.10
{% endhighlight %}

To access that from Java, you need to create a new class for each level of nesting. In this case, you would need to create a new static inner class for the `remoteHost` field and include all of it's sub-fields as instance variables. The following is the updated `ApplicationProperties` class.

{% highlight java %}
@ConfigurationProperties("application")
public class ApplicationProperties {

    private String emailServer;
    private String serviceUser;

    @NotNull
    private String servicePassword;

    private RemoteHost remoteHost;

    //getters and setters

    public static class RemoteHost {

        private String hostName;

        //getters and setters
    }
}
{% endhighlight %}

Spring Boot will take care of instantiating the inner class. In your application code, you can access the newly added property by calling `applicationProperties.getRemoteHost().getHostName()`.

## Easy refactoring

Using `@ConfigurationProperties` allows you to manage configuration values in a type-safe manner. Refactoring is made easier for you since you have a single class where all property values are defined. Consider the `@Value` annotation for a moment. Injecting a property using `@Value` requires you to [specify the property as a string](https://tuhrig.de/why-using-springs-value-annotation-is-bad/ "Why using Spring’s @Value annotation is bad"). What if you have multiple places in your codebase where this property is used? If your IDE is not smart enough, you would need to do a project wide search and replace when renaming a single property in the properties file.

## Farewell

I hope you gained some new ideas reading this. A class holding all configuration values has its merits, namely, type-safety, advantages in refactoring and validation can be in one place. But don't go and start replacing all of your `@Value` annotations with configuration classes just yet. Use common sense. Sometimes it is less work to just use `@Value` or Spring's `Environment` abstraction.
