---
layout: post
title: "Configuring Tomcat to accept Estonian ID card"
excerpt: "In order to accept Estonian ID card, Tomcat needs to be configured to accept client side certificates."
modified: 2013-09-20 18:17:21 +0300
categories: articles
tags: [apache, tomcat, esteid, client certificates, ssl, tls, java, pki, crl, ocsp]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

In order to accept Estonian ID card, Tomcat needs to be configured to accept client side certificates. But before that, [SSL needs to be configured]({{site.url}}/articles/configure-ssl-on-apache-tomcat "Configure SSL on Apache Tomcat").

## Setting up client side authentication

First you need all SK's root certificates.

> SK (Certification Centre, legal name AS Sertifitseerimiskeskus) is Estonia's primary and currently the only certification authority (CA), providing certificates for authentication and digital signing to national identity documents (ID-card, residence permit, Digi-ID, Mobiil-ID).

{% highlight bash %}
$ wget http://www.sk.ee/crls/esteid/esteid2007.crl
$ wget http://www.sk.ee/crls/juur/crl.crl
$ wget http://www.sk.ee/crls/eeccrca/eeccrca.crl
$ wget http://www.sk.ee/repository/crls/esteid2011.crl
{% endhighlight %}

Add these certificates to your keystore.

{% highlight bash %}
$JAVA_HOME/bin/keytool -importcert -file "/path/to/certs/EECCRCA.pem.cer" -alias eeccrca
$JAVA_HOME/bin/keytool -importcert -file "/path/to/certs/ESTEID-SK 2007.PEM.cer" -alias esteid2007
$JAVA_HOME/bin/keytool -importcert -file "/path/to/certs/ESTEID-SK 2011.pem.cer" -alias esteid2011
$JAVA_HOME/bin/keytool -importcert -file "/path/to/certs/JUUR-SK.PEM.cer" -alias juur
{% endhighlight %}

For each import, you will be asked for the keystore password and your confirmation on whether you trust this certificate or not.

Modify your SSL connector element in the `server.xml` file.

{% highlight xml %}
<Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
               maxThreads="150" scheme="https" secure="true"
               clientAuth="true" sslProtocol="TLS"
               keystoreFile="/path/to/.keystore" keystorePass="changeit"
               truststoreFile="/path/to/.keystore"
               truststorePass="changeit" />
{% endhighlight %}

You need to change `clientAuth` to true and set the `truststoreFile` path and password. That's all for Tomcat configuration. Now you need to enable client side authentication from your applicaiton. The following is a sample `web.xml`:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                  http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
           version="3.0">

  <security-constraint>
    <web-resource-collection>
      <web-resource-name>Everything</web-resource-name>
      <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <user-data-constraint>
      <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>
  </security-constraint>
  <login-config>
    <auth-method>CLIENT-CERT</auth-method>
  </login-config>

</web-app>
{% endhighlight %}

If you wish to get information about the client or server certificates in your application, then you can use the following snippet.

{% highlight java %}
//omitting imports

@WebServlet("/*")
public class IdCardServlet extends HttpServlet {
  protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
  }

  protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    PrintWriter writer = response.getWriter();
    X509Certificate[] certs = (X509Certificate[]) request.getAttribute("javax.servlet.request.X509Certificate");
    if (certs != null) {
      List<X509Certificate> certificateList = Arrays.asList(certs);
      for (X509Certificate cert : certificateList) {
        writer.write(cert.getSubjectDN().getName());
      }
    }
  }
}
{% endhighlight %}

At this point you're able to read client side certificates in your web app. The only thing left to do is to make sure the certificate presented by the client has not revoked by the CA. To accomplish that you're going to have to keep a **certificate revocation list (CRL)** or make an **OCSP (online certificate status protocol)** request. These topics will be discussed in a future article.
