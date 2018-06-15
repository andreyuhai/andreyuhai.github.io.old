---
layout: post
title: "How to access phpmyadmin after installation?"
excerpt: "You've just installed phpmyadmin from your package manager. Here's what you need to do in order to access it"
modified: 2013-04-27 19:46:11 +0300
categories: articles
tags: [phpmyadmin, php, ubuntu, how-to, apache]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
aging: true
---

You've just installed phpmyadmin from your package manager and want to access it. Here's what you need to do. I've used Ubuntu in this example so the paths might vary from distro to distro.

You need to add phpmyadmin's conf to `apache2.conf`.

{% highlight bash%}
$ sudo echo "Include /etc/phpmyadmin/apache.conf" >> /etc/apache2/apache2.conf
{% endhighlight %}

All that needs to be done now is to restart apache.

{% highlight bash%}
$ sudo /etc/init.d/apache2 restart
{% endhighlight %}

Point your browser to [http://localhost/phpmyadmin](http://localhost/phpmyadmin "phpmyadmin localhost") and you should be presented with phpmyadmin's login page.
