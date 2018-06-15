---
layout: post
title: "Configure SSL on Apache Tomcat"
excerpt: "Here we are going to have a look at how to set up SSL on Apache Tomcat 7.0."
modified: 2013-09-03 21:37:14 +0300
categories: articles
tags: [SSL, Apache, Tomcat, configuration, how-to]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
aging: true
---

Here we are going to have a look at how to set up SSL on Apache Tomcat 7.0. First of all, you must create a keystore file. This stores the server's private key and certificate. This can be achieved using the keytool bundled with Java.

{% highlight bash %}
$JAVA_HOME/bin/keytool -genkey -alias mydomain -keyalg RSA -keystore /path/to/keystore
{% endhighlight %}


Enter a password for the keystore. Then enter information about the certificate. You are prompted for the following information.

<pre>
Enter keystore password:  
Re-enter new password:
What is your first and last name?
  [Unknown]:  John Smith
What is the name of your organizational unit?
  [Unknown]:  John's Diner
What is the name of your organization?
  [Unknown]:  John's Diner
What is the name of your City or Locality?
  [Unknown]:  Paris
What is the name of your State or Province?
  [Unknown]:  Île-de-France
What is the two-letter country code for this unit?
  [Unknown]:  fr
Is CN=John Smith, OU=John's Diner, O=John's Diner, L=Paris, ST=Île-de-France, C=fr correct?
  [no]:  yes

Enter key password for mydomain:
</pre>

This creates a new keystore file and a self-signed certificate. The location is determined by the `-keystore` flag. Each certificate in a Java keystore is associated with a unique alias.

The next step is editing Tomcat's `server.xml` located at `$CATALINA_BASE/conf/server.xml` (the location of this file varies depending on your operating system). A new connector needs to be added. Find the location where previous connectors are defined and add the following connector. If you look carefully, then you might find a commented SSL connector. You can uncomment it and modify it so it matches the following example.

{% highlight xml %}
<Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
           maxThreads="150" scheme="https" secure="true"
           clientAuth="false" sslProtocol="TLS"
           keystoreFile="/path/to/keystore"
           keystorePass="password" />
{% endhighlight %}

Restart Tomcat and you should be in business. Point your browser to [https://localhost:8443](https://localhost:8443) and you should get an SSL warning because you are using a self-signed certificate. If nothing happens, then check the `catalina.out` log for more information.

**References**

* [SSL Configuration HOW-TO](http://tomcat.apache.org/tomcat-7.0-doc/ssl-howto.html)
