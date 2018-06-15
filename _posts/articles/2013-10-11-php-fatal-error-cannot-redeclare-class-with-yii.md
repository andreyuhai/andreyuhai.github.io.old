---
layout: post
title: "PHP Fatal error:  Cannot redeclare class with Yii"
excerpt: "I am fairly new to PHP and Yii. Lately i struggled with a 'cannot redeclare class' error."
modified: 2014-06-10 23:37:47 +0300
categories: articles
tags: [php, yii, error, codeception]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
aging: true
---

I am fairly new to PHP and Yii. Lately i struggled with a "cannot redeclare class" error. Here's a brief overview of my setup. I'm using Yii's Active Record and I have 2 models (`ParentModel` and `ChildModel`) with a one-to-many relation. Both model classes have namespaces. In addition, I'm using [Codeception](http://codeception.com/) for testing. Upon running unit tests, i received the following error:

<pre>
PHP Fatal error:  Cannot redeclare class application\models\ChildModel
in /var/www/demo/app/models/ChildModel.php on line 49
</pre>


This occurred while i was trying to save `ParentModel` to database. Doing the same process while using the web application did not cause this error. After hours of debugging, reading Yii's source code and googling, I finally found a solution. The culprit was the usage of namespaces in model classes. Here's how I defined the relations between the two model classes:

{% highlight php startinline=true %}

  public function relations() {
    return array(
      'childModels' => array(self::HAS_MANY, 'ChildModel', 'parent_model_id')
    );
  }

{% endhighlight %}

{% highlight php startinline=true %}

  public function relations() {
    return array(
      'parentModel' => array(self::BELONGS_TO, 'ParentModel', 'parent_model_id')
    );
  }

{% endhighlight %}

**Solution**

As mentioned previously, the problem was the usage of namespaces, or the mis-usage of namespaces, to be more precise. In the relations array I should have used the model's fully qualified name (namespace + class name). The following is the working code:

{% highlight php startinline=true %}

  public function relations() {
    return array(
      'childModels' => array(self::HAS_MANY, 'application\models\ChildModel', 'parent_model_id')
    );
  }

{% endhighlight %}

{% highlight php startinline=true %}

  public function relations() {
    return array(
      'ParentModel' => array(self::BELONGS_TO, 'application\models\ParentModel', 'parent_model_id')
    );
  }

{% endhighlight %}


Hopefully this post was helpful for somebody.
