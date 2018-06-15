---
layout: post
title: "XsltException caused by JRebel"
excerpt: "XsltException caused by a seemingly missing stylesheet. Turns out JRebel was the culprit."
modified: 2013-04-26 20:56:49 +0300
categories: articles
tags: [java, jrebel, windows, maven, pom.xml, rebel.xml]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
aging: true
---

I stumbled upon a problem where a Java web application complained about an XsltException which was caused by a missing stylesheet. Although the stylesheet existed, it was not found for some reason.

After configuring log4j to see, where the app was looking for the stylesheet, I realised that it was being looked for from the wrong location. The cause of the problem was Maven and JRebel. The following is a snippet from the pom.xml file.

{% highlight xml %}
<webResources>
  <resource>
    <directory>${basedir}/resources/${conf.name}</directory>
  </resource>
  <resource>
    <directory>${basedir}/../build/ear/web</directory>
  </resource>
</webResources>
{% endhighlight %}

The issue was that these resource tags were in reverse order in `rebel.xml` and that caused the application to look for stylesheets from the wrong location. As a side note, this issue was seen only on Windows environments, Linux and OSX were fine. To fix it, I changed the order in my `pom.xml` file.
