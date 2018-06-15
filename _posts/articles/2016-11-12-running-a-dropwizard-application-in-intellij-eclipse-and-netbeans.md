---
layout: post
title: "Running a Dropwizard application in IntelliJ IDEA, Eclipse and NetBeans"
excerpt: In addition to executing a Dropwizard application from the command line, you can increase your productivity by configuring your IDE to do that for you. This post is going to be looking at how to run a Dropwizard application in IntelliJ IDEA, Eclipse and NetBeans.
modified: 2016-11-12 16:33:22 +0200
categories: articles
tags: [intellij, eclipse, netbeans, dropwizard, java]
image:
  feature: 2016-11-09-running-a-dropwizard-application-inside-intellij-eclipse-and-netbeans/cover.jpg
  credit: Farzad Nazifi
  creditlink: https://unsplash.com/photos/p-xSl33Wxyc
comments: true
share: true
published: true
aging: true
---

[Dropwizard](http://www.dropwizard.io) is a framework for building [RESTful](https://www.sitepoint.com/what-does-restful-really-mean/) web services in Java. Similar to [Spring Boot](https://projects.spring.io/spring-boot/), Dropwizard applications are packaged into an executable *fat* JAR file. In addition to executing a Dropwizard application from the command line, you can increase your productivity by configuring your IDE to do that for you. I'm going to be looking at how to run a Dropwizard application in [IntelliJ IDEA](https://www.jetbrains.com/idea/), [Eclipse](https://eclipse.org/downloads/) and [NetBeans](https://netbeans.org/).

To execute a Dropwizard application, its `main` method needs to be called with the correct arguments. When you execute a JAR file from the command line, the JVM reads the entry point to your application from the `MANIFEST.MF` file. It contains key-value pairs and the key we're interested in is `Main-Class`. In a Dropwizard application, this is set to your `Applicaiton` class.

Equipped with this knowledge, it should be relatively easy to instruct your IDE to run a Dropwizard application during development. I'm going to be looking at how to do that in the three most popular Java IDEs based on [Zeroturnaround's Java Tools and Technologies Landscape Report 2016](http://zeroturnaround.com/rebellabs/java-tools-and-technologies-landscape-2016/ "Java Tools and Technologies Landscape Report 2016")

## IntelliJ IDEA

1. Open *Run/Debug Configuration* menu by selecting Run -> Edit Configurations... from the menu.
2. Add a new configuration by pressing Alt+Insert or clicking the green plus sign in the top left corner.
3. From the dropdown menu, select *Application*.
4. Add a name to the configuration.
5. Enter the fully qualified name of your main class.
6. Enter `server config.yml` to the *Program arguments* field. These are the same arguments you would pass to your application if you were running it from the command line (view image below).
7. Press Shift+F10 or select *Run* -> *Run \<your run configuration name\>* from the menu to start your Dropwizard application

<figure>
	<a href="{{ site.url}}/images/2016-11-09-running-a-dropwizard-application-inside-intellij-eclipse-and-netbeans/idea_conf.png" class="image-popup"><img src="{{ site.url}}/images/2016-11-09-running-a-dropwizard-application-inside-intellij-eclipse-and-netbeans/idea_conf.png" alt="IntelliJ IDEA run configuration menu"></a>
	<figcaption>IntelliJ IDEA run configuration menu</figcaption>
</figure>

## Eclipse

1. Open your project's *Run Configurations* menu by selecting *Run* -> *Run Configurations*.
2. Create a new Java Application launch configuration.
3. On the *main* tab choose a name for the configuration and specify the main class.
4. On the *arguments* tab add `server config.yml` to the program arguments text field.
5. Click the *run* button or press Ctrl+F11 to run your Dropwizard application.

<figure class="half">
  <a href="{{ site.url }}/images/2016-11-09-running-a-dropwizard-application-inside-intellij-eclipse-and-netbeans/eclipse_main.png" class="image-popup">
	  <img src="{{ site.url }}/images/2016-11-09-running-a-dropwizard-application-inside-intellij-eclipse-and-netbeans/eclipse_main.png" alt="Main tab of the Run Configuration window">
  </a>
  <a href="/images/2016-11-09-running-a-dropwizard-application-inside-intellij-eclipse-and-netbeans/eclipse_arguments.png" class="image-popup">
	  <img src="/images/2016-11-09-running-a-dropwizard-application-inside-intellij-eclipse-and-netbeans/eclipse_arguments.png" alt="Arguments tab of the Run Configuration window">
  </a>
	<figcaption>Eclipse's run configuration menu</figcaption>
</figure>

## NetBeans IDE

1. Open project properties by right-clicking on the project folder in the file explorer and select *properties*.
2. Select the *Run* category from the left.
3. Enter the fully qualified name of your main class.
4. Enter `server config.yml` to the *Arguments* field and click *OK*.
5. Now you should be able to run your Dropwizard application by pressing F6 or by selecting *Run* -> *Run Project* from the menu.

<figure>
	<a href="{{ site.url}}/images/2016-11-09-running-a-dropwizard-application-inside-intellij-eclipse-and-netbeans/netbeans.png" class="image-popup"><img src="{{ site.url}}/images/2016-11-09-running-a-dropwizard-application-inside-intellij-eclipse-and-netbeans/netbeans.png" alt="NetBeans' project properties"></a>
	<figcaption>NetBeans' project properties</figcaption>
</figure>

## Final words

If the IDE you use is not listed here, you can still apply the same principles. The key is to instruct the IDE to call the `main` method with the desired arguments. Following the same steps it is very easy to attach a debugger as well. Instead of selecting the *run* option, you need to start the applicaiton in *debug* mode. For instance, in IntelliJ, instead of pressing Shift+F10 to run the application, press Shift+F9. In Eclipse and NetBeans the key combinations are F11 and Ctrl+F5 respectively.
