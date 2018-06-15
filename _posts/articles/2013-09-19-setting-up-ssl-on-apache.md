---
layout: post
title: "Setting up SSL on Apache"
excerpt: "This article explains how to create an SSL certificate, get it signed and configure Apache web server"
modified: 2013-09-19 17:24:19 +0300
categories: articles
tags: [apache, ssl, tls, pki, sni, openssl]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

**Disclaimer:** these instructions were tested on Ubuntu 13.04 and Apache 2.2.22.

Setting up SSL on the Apache web server consists of multiple steps. Firstly, a key pair must be generated. Based on that, a certificate signing request (CSR) can be made. And lastly, after a valid CA has issued an SSL certificate, Apache web server needs to be configured.

## 1. Generate public and private key

Generate keys using openSSL

{% highlight bash %}
$ openssl genrsa -out server.key 1024
{% endhighlight %}

## 2. Generate CSR

CSR stands for Certificate Signing Request.

{% highlight bash %}
$ openssl req -new -key server.key -out server.csr
{% endhighlight %}

You are about to be asked to enter information that will be incorporated into your certificate request. What you are about to enter is what is called a Distinguished Name or a DN. There are quite a few fields but you can leave some blank. For some fields there will be a default value, If you enter '.', the field will be left blank.

The following are the questions being asked.

<pre>
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
Please enter the following 'extra' attributes to be sent with your certificate request

A challenge password []:
An optional company name []:
</pre>

**The Common Name should be your server's DNS name.**

## 3. Order SSL certificate

Now that you have your certificate signing request, it is time to order an SSL certificate from your favorite public certificate authority (CA).

For development purposes you can generate a self-signed certificate, but it is highly recommended not to go that route for production environments.

{% highlight bash %}
$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
{% endhighlight %}

## 4. Configuring Apache web server

Move `server.key` and `server.crt` to `/etc/apache2/ssl/key` and `/etc/apache2/crt/` respectively. If these directories don't exist, then create them (in Linux distros other than Ubuntu, the location of Apache might vary).

To enable SSL on Apache, the SSL module must be enabled. This can be achieved with the following command.

{% highlight bash %}
$ a2enmod ssl
{% endhighlight %}

Now you need to restart Apache for the changes to take effect.

{% highlight bash %}
$ /etc/init.d/apache2 restart
{% endhighlight %}

Enable Apache to listen on port 443. To achieve that, you need to modify the ports.conf file which exists at /etc/apach2/ports.conf. If it does not already exist, then add the following line to ports.conf: `Listen 443`

Create a new virtualhost. To begin with, a new site configuration needs to be added.

{% highlight bash %}
$ nano /etc/apache2/sites-available/mysite
{% endhighlight %}

Add a basic ssl virtualhost setup

{% highlight xml %}
NameVirtualHost *:443

<VirtualHost *:443>

    DocumentRoot /var/www/mysite
    ServerName mysite.com

    SSLEngine on

    SSLCertificateFile /etc/apache2/crt/server.crt
    SSLCertificateKeyFile /etc/apache2/ssl/key/server.key
</VirtualHost>
{% endhighlight %}

Make sure you have `mysite` folder created at `/var/www/`. If not, then create a new folder and add some test content to it.

{% highlight bash %}
$ mkdir /var/www/mysite
$ echo -e "Hello world, SSL works" | sudo tee /var/www/mysite/index.html
{% endhighlight %}

And enable the site

{% highlight bash %}
$ a2ensite mysite
{% endhighlight %}

Restart Apache and SSL should work

{% highlight bash %}
$ /etc/init.d/apache2 restart
{% endhighlight %}

If you are running a web server on localhost, then make sure you modify your hosts file to point mysite.com to your machine. If that has been dealt with, point your browser to [https://mysite.com/](https://mysite.com/).

## 5. Server Name Indication

Server Name Indication (SNI) is an extension to the TLS protocol that indicates what hostname the client is attempting to connect to at the start of the handshaking process. This allows a server to present multiple certificates on the same IP address and port number and hence allows multiple secure (HTTPS) websites to be served off the same IP address without requiring all those sites to use the same certificate. It is the conceptual equivalent to HTTP/1.1 virtual hosting for HTTPS.

To make use of SNI, it is necessary that the vast majority of users use web browsers that support it. Users whose browsers do not support SNI will be presented with a default certificate and hence are likely to receive certificate warnings, unless the server is equipped with a wildcard certificate that matches the name of the website. As of November 2012, the only major user bases whose browsers do not support SNI appear to be users of Android 2.x (default browser), Internet Explorer (any version) on Windows XP and versions of Java before 1.7 on any operating system. If your service has users on these platforms, then it is required to run only one HTTPS site/service per IP.

## References
* [http://en.wikipedia.org/wiki/Server_Name_Indication](http://en.wikipedia.org/wiki/Server_Name_Indication).
