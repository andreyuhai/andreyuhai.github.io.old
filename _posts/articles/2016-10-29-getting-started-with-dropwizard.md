---
layout: post
title: "Getting started with Dropwizard"
excerpt: Dropwizard is a framework for building RESTful web services in Java. In this tutorial we're going to have a look at how to get started with developing a Dropwizard application by building a new service from scratch.
modified: 2016-10-29 10:36:38 +0300
categories: articles
tags: [dropwizard, rest, java, web service, jersey, jackson]
image:
  feature: 2016-10-27-getting-started-with-dropwizard/cover.jpg
  credit: gunshowcomic.com
  creditlink: http://gunshowcomic.com/316
comments: true
share: true
published: true
aging: true
canonical-url: https://www.sitepoint.com/tutorial-getting-started-dropwizard/
---
*This article was originally published on [SitePoint](https://www.sitepoint.com/tutorial-getting-started-dropwizard/) on October 13, 2016. For more interesting content about Java, check out [SitePoint's Java channel](https://www.sitepoint.com/java/ "Java channel at SitePoint").*

[Dropwizard](http://www.dropwizard.io) is a framework for building [RESTful](https://www.sitepoint.com/what-does-restful-really-mean/) web services in Java. In this tutorial we're going to have a look at how to get started with developing a Dropwizard application by building a new service from scratch. This article is going to cover the very basics. The goal is to implement and run a Dropwizard application which is able to respond to an HTTP GET request.

After a short introduction to Dropwizard and our application domain, we will see how to create a new project, talk about its structure, and configure it. Later on we will implement a simple RESTful API  before finally building and running your application.

The sample project is available on [GitHub](https://github.com/indrekots/events-service "Events service Github repository"). Key commits are tagged and I'll be referencing them throughout this post after each major change. If you feel that something is unclear and wish to see a snapshot of the whole project, feel free to [check out](https://git-scm.com/docs/git-checkout "Git checkout documentation") the referenced tag.

The only prerequisites you need to have installed are Maven, JDK, and a text editor or IDE of your choice.

## What is Dropwizard?

As already stated, Dropwizard is a framework for building RESTful web services in Java. In essence, it is a *glue* framework which bundles together popular and battle-tested Java libraries and frameworks to make it easier to start building new RESTful web services.

Here's an incomplete list of some of the libraries that Dropwizard uses.

* [Jersey](https://jersey.java.net/) - reference implementation of [JAX-RS](https://jax-rs-spec.java.net/), the API for RESTful web services in the Java platform
* [Jackson](http://wiki.fasterxml.com/JacksonHome) - library for processing JSON data
* [Jetty](https://eclipse.org/jetty/) - HTTP server and Java Servlet container
* [Metrics](http://metrics.dropwizard.io/) - library for capturing JVM- and application-level metrics for monitoring
* [Guava](https://github.com/google/guava) - utility library
* [Logback](http://logback.qos.ch/) - logging library

## Introducing our demo application domain

To get a better understanding of Dropwizard, we're going to build a RESTful web service from the ground up. It is going to be a back-end for a hypothetical events app. Imagine you've moved to a new city and would like to plan your evening. This app will list events based on your search criteria and location. The back-end should be able to provide a list of events. It should also be possible to add new events and update existing events. To keep things simple, we're going to exclude user authentication and authorization for now.

## Project structure

I'm going to use [Maven](https://maven.apache.org/) as a build tool. In my opinion, the easiest way to get started with a new application is to create a project using the [`dropwizard-archetype`](https://github.com/dropwizard/dropwizard/tree/master/dropwizard-archetypes). [Archetype](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html "Maven Archetype") is essentially a tool for bootstrapping a Maven project from a template.

Run the following command in your terminal to get started with a new Dropwizard project.

{% highlight bash %}
mvn archetype:generate \
	-DarchetypeGroupId=io.dropwizard.archetypes \
	-DarchetypeArtifactId=java-simple \
	-DarchetypeVersion=1.0.2
{% endhighlight %}

As of October 2016, the latest version of Dropwizard is 1.0.2. Be sure to check the most recent [version](https://mvnrepository.com/artifact/io.dropwizard/dropwizard-core) and update `archetypeVersion` accordingly. Running the `archetype:generate` command prompts you to enter some project specific fields (e.g. groupId, artifactId). When entering the project name, keep in mind that this field is used to generate application and configuration classes. Use CamelCase and do not insert spaces to make sure that the generated class names are going to be valid.

For example, I named the application *Events*. Therefore the generated classes are going to be named `EventsApplication` and `EventsConfiguration`. If I had used *events-app* as the name, `event-appApplication` and `event-appConfiguration` would have been generated. Unfortunately, these are invalid class names and the compiler will point that out to you.

On successful completion, This is what the generated folder structure looks like.

{% highlight bash %}
.
├── config.yml
├── pom.xml
├── README.md
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── mydomain
    │   │           ├── api
    │   │           ├── cli
    │   │           ├── client
    │   │           ├── core
    │   │           ├── db
    │   │           ├── EventsApplication.java
    │   │           ├── EventsConfiguration.java
    │   │           ├── health
    │   │           └── resources
    │   └── resources
    │       ├── assets
    │       └── banner.txt
    └── test
        ├── java
        │   └── com
        │       └── mydomain
        │           ├── api
        │           ├── client
        │           ├── core
        │           ├── db
        │           └── resources
        └── resources
            └── fixtures
{% endhighlight %}

Application configuration is stored in `config.yml`. We're going to look at it in more detail in the next paragraph. Most of the meat is located in `src/main/java` including the two aforementioned classes - `EventsApplication` and `EventsConfiguration`.

The following is an overview of which files should go into which folder:

* `api` - Here live your application representations, which are essentially the entities of your API. They are serialized and deserialized to JSON.
* `cli` - Contains commands that can be executed from the command line. They are not covered in this tutorial.
* `client` - HTTP client implementation of your API. For easy consumption of your API you can implement a client and place it here. We're not going to do it in this tutorial.
* `core` - This is where you implement your domain logic.
* `db` - Classes related to database access.
* `health` - Contains health checks. These are runtime tests which verify whether your application is behaving correctly.
* `resources` - Jersey resources.

We're not going to use all of these packages since I'd like to keep the tutorial on the easier side. Dropwizard isn't strict about the package naming scheme. If you wish you can rename and restructure the package hierarchy to suit your needs better.

The initial project structure can be seen by checking out the `starting-point` tag.

## Application configuration

Storing variables which might vary depending on the execution environment in a configuration file is considered to be a good practice. When running your application, Dropwizard expects to receive a configuration file as a command line argument. By default configuration files should be written in [YAML](http://yaml.org/ "Official YAML website"). The parsed configuration fields are used to build a new instance of a configuration class.

For now, let's keep the configuration file as straightforward as possible. There are two things I would like add. Firstly, let's change the root path of our application so all REST endpoints will be served from `/api/*`. The `rootPath` parameter is one of many [built-in configuration parameters](http://www.dropwizard.io/1.0.2/docs/manual/configuration.html "Example configuration file with build-in configuration parameters") Dropwizard has to offer, so we do not need to change any code to have it available.

Secondly, since we're dealing with events, we care about the start time, which can be [represented in many ways](https://en.wikipedia.org/wiki/Category:Date_and_time_representation_by_country "Date and time representations by country"). Let's add a date format configuration parameter, which enables us to control the output date format from a configuration file.

First, modify `config.yml` and add the following configuration.

{% highlight yaml %}
server:
  rootPath: /api/*
dateFormat: yyyy-MM-dd'T'HH:mmZ
{% endhighlight %}

Of course we need to access the custom configuration parameter `dateFormat` from within the running application. When Dropwizard is started, configuration fields are mapped to a configuration class' member fields.

So let's edit our configuration class and include the `dateFormat` field. Open `EventsConfiguration` and add the following lines:

{% highlight java %}
@NotEmpty
private String dateFormat;

public String getDateFormat() {
    return dateFormat;
}
{% endhighlight %}

An application shouldn't trust an external configuration file to be correct. To help validate its correctness, we can use [Hibernate Validator](http://hibernate.org/validator/ "About Hibernate Validator"). The `@NotEmpty` annotation makes sure the string is not empty. Otherwise Dropwizard fails to start.

Keep in mind that this does not check if the date format pattern is actually valid. It only checks if it is defined in the configuration file. You could create your own custom validator which will print out a nice error message at start-up if the pattern is invalid. To keep things simpler, I'm not going to tackle this issue at the moment.

Check out `app-configuration` tag to see the project after configuration changes.

## Application class

This is what the generated application class looks like.

{% highlight java %}
public class EventsApplication extends Application<EventsConfiguration> {
    public static void main(final String[] args) throws Exception {
        new EventsApplication().run(args);
    }

    @Override
    public void run(EventsConfiguration configuration, Environment environment) {
    }

    @Override
    public void initialize(Bootstrap<EventsConfiguration> bootstrap) {
    }
}
{% endhighlight %}

When starting the service, the `main` method inside the `EventsApplication` class gets invoked. This is the entry point for a Dropwizard application. A new `EventsApplication` object is created and its `run` method is called. The `run` method is used to configure Dropwizard's environment.

This is the place to apply the date pattern used in our application. For that we configure Jackson's `ObjectMapper`, which will later (de)serialize the request and response parameters from Java classes to JSON and vice versa. Let's modify its date format by explicitly creating a new instance of `SimpleDateFormat` and passing it the pattern from the configuration object.

{% highlight java %}
@Override
public void run(EventsConfiguration configuration,
                Environment environment) {
    DateFormat eventDateFormat = new SimpleDateFormat(configuration.getDateFormat())
    environment.getObjectMapper().setDateFormat(eventDateFormat);
}
{% endhighlight %}

Later we're going to modify the `run` method to register new *resources*.

The application class has one more method we need to look at although we're not going to add any code inside it: `initialize` is used to go through the application’s bootstrapping phase. Here you can add [bundles](http://www.dropwizard.io/1.0.2/docs/manual/core.html#bundles) and [commands](http://www.dropwizard.io/1.0.2/docs/manual/core.html#commands) for example. Bundles are groups of reusable functionality - think of them as add-ons to your application. Commands can be used to create additional functionality that can be accessed from the command line.

Checkout `date-format-configuration` tag to see the project after changes made in the application class.

## Creating a representation of domain objects

Representations are the entities of your API. They *represent* the current or the desired state of a resource. For example, when a client makes an HTTP GET request to the *events* resource, the server sends back a representation of the current state of events. Representations can be serialized to any format but we're going to use JSON in this tutorial.

Let's think about what fields we want to include in the events representation class. An event should have a name. Also, we need to know when and where the event is taking place. Hosts might want to add additional information about the event. This calls for a description field. It is a good idea to include an id as well since we want to make sure each event is uniquely identifiable.

Create a new `Event` class in `com.mycompany.api` (or whatever package naming scheme you are using).

{% highlight java %}
public class Event {
    private long id;
    private String name;
    private String description;
    private String location;
    private Date date;

    //getters and setters omitted
}
{% endhighlight %}

In most cases using a string to represent a location might not be enough. You could create a `Location` class which can hold more detailed information (e.g. latitude, longitude). Dropwizard will make sure nested objects are serialized correctly.

Check out `event-representation` tag to see the project after creating a new representation class.

## Create a simple resource

Under the hood, Dropwizard uses [Jersey](https://jersey.java.net/) to build RESTful web services. Jersey is a reference implementation of [JAX-RS](https://jax-rs-spec.java.net/) ([JSR 311](https://jax-rs-spec.java.net/) & [JSR 339](https://jcp.org/aboutJava/communityprocess/final/jsr339/index.html)). A lot goes on in implementing resources and I don't intend for this article to be a comprehensive overview of Jersey. If you wish to learn more, feel free to [browse Jersey's documentation](https://jersey.java.net/documentation/latest/getting-started.html).

To get started with Jersey, let's create a resource for events. Additionally, we need to define a URL which can be used to access the resource and retrieve its current state.

Let's create a very simple implementation of the events resource. The goal is to expose a URL for retrieving all events. For now we're going to return dummy data. Create a new class called `EventResource` in the `resources` package.

{% highlight java %}
@Path("events")
@Produces(MediaType.APPLICATION_JSON)
public class EventResource {

    @GET
    public List<Event> allEvents() {
        Event e = new Event();
        e.setDate(new Date());
        e.setName("Birthday");
        e.setId(1L);
        e.setDescription("Please be on time!");
        e.setLocation("221B Baker Street");

        return Collections.singletonList(e);
    }
}
{% endhighlight %}

The `@Path` annotation specifies the URL root for this resource. In the next line we specify the representation format. Representation classes are serialized to JSON and returned to the client. `EventResource` at its current state has only one method: `allEvents`. It is annotated with the HTTP verb it should respond to. So for example, when a GET request to `/events` is received by Dropwizard, `allEvents` gets invoked. In REST speak, GET is used to retrieve a representation of the current state of a resource.

Inside the method a new `Event` object is instantiated with dummy data. The event object is wrapped in a list which is then returned. I'm using [`singletonList`](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#singletonList-T-) from the Collections class for convenience. Creating a new list manually and adding the element to the list would work perfectly as well.

To see the project after resource creation, check out `event-resource` tag.

## Registering a resource

By default Dropwizard doesn't know about resource classes and we need to tell it explicitly which resources to expose. Technically speaking, resources need to be registered with Dropwizard's Jersey Environment. As mentioned earlier, resource registration can be done in the `EventsApplication` class' `run` method.

Modify it to look like the following example.

{% highlight java %}
@Override
public void run(EventsConfiguration configuration,
    		Environment environment) {
    environment.getObjectMapper().setDateFormat(new SimpleDateFormat(configuration.getDateFormat()));

    EventResource eventResource = new EventResource();
    environment.jersey().register(eventResource);
}
{% endhighlight %}

The current state of the project can be seen by checking out `register-resource` tag.

## Building the application

To run a Dropwizard application, it needs to be built first. It is recommended that you [build your application as a *fat JAR file*](http://www.dropwizard.io/1.0.2/docs/getting-started.html#building-fat-jars). This is a JAR file that contains the classes from your project and all the libraries it depends on, so everything the application needs is bundled into a single file, which makes it easy to deploy and run. The same artifact can be promoted from test to staging to production while being sure there's no version changes in libraries used.

If you followed the article and created a new project with [`dropwizard-archetype`](https://github.com/dropwizard/dropwizard/tree/master/dropwizard-archetypes) then Maven is already configured to build a fat JAR file using the [maven-shade-plugin](https://maven.apache.org/plugins/maven-shade-plugin/ "Apache Maven Shade Plugin").

Simply run the following command in the root of your project (the folder where `pom.xml` is located).

{% highlight bash %}
$ mvn package
{% endhighlight %}

## Running the application

You can now execute the jar file as follows.


{% highlight bash %}
$ java -jar target/events-0.0.1-SNAPSHOT.jar server config.yml
{% endhighlight %}

For successful execution, we need to pass two command line arguments to Dropwizard. The first one, `server`, instructs Dropwizard to start an HTTP server and run the service. The second argument is a configuration file which is required by the server command.

If the app started without any errors, point your browser (or a command line http tool such as [curl](https://curl.haxx.se "command line tool and library for transferring data with URLs")) to `http://localhost:8080/api/events`. The server should return with a valid response containing the following message body.

{% highlight json %}
[{"id":1,"name":"Birthday","description":"Please be on time!","location":"221B Baker Street","date":"2016-10-03T12:09+0200"}]
{% endhighlight %}

## Summary

By now you should have a basic idea what Dropwizard is about. With the help of existing libraries, Dropwizard enables developers to easily create new RESTful web services.

There's a lot we did not cover. First of all, the requirements we set for our application are not met. At the moment, only dummy data is returned to the client. There's going to be a follow-up article where we're going to add additional features to the Events service and look into parts of Dropwizard we were not able to cover in this article.
