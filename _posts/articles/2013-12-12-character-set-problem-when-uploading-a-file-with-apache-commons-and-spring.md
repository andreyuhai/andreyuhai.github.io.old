---
layout: post
title: "Character set problem when uploading a file"
excerpt: "Non-ASCII characters in file name get garbled when uploading a file with Apache Commons and Spring CommonsMultipartResolver."
modified: 2013-12-12 16:58:01 +0300
categories: articles
tags: [spring, java, ascii, apache commons, utf-8]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

When posting a form as multipart/form-data, I encountered a situation where UTF-8 characters got garbled. I was using Spring's *CommonsMultipartResolver*. The trick that solved it for me was to set the character set for multipartresolver bean.

{% highlight xml %}
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<property name="defaultEncoding" value="UTF-8"/>
</bean>
{% endhighlight %}
