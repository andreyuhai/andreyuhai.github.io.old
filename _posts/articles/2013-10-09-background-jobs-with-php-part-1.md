---
layout: post
title: "Background jobs with PHP part 1"
excerpt: "A background job is a task that is executed 'behind the scenes' and without user intervention."
modified: 2013-10-09 14:28:17 +0300
categories: articles
tags: [php, background processing, cron, unix, composer, redis, resque]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
aging: true
---

A background job is a task that is executed "behind the scenes" and without user intervention. In a web application context, it is a task that is executed outside of the main flow. Therefore the end user has no clue of it ever happening. This is great, because you don't want to make your users wait for a computationally heavy task to finish and then load a web page. A more elegant solution would be to load the page as soon as possible, and run the time consuming tasks, which are not required for displaying the page, in the background. In addition, a background job might not even be user initiated. These include scheduled and recurring tasks.

This series of posts covers some of the solutions for implementing a background job in PHP. Firstly, let's look at how to create jobs that are not initiated by the user. The easiest method might be to use cron.

>The software utility cron is a time-based job scheduler in Unix-like computer operating systems.

On Windows you can use the [Task Scheduler](http://en.wikipedia.org/wiki/Windows_Task_Scheduler) but this is not going to be covered in this post.

## Cron jobs

Cron is a daemon which executes commands at a specified point in time. To add a new job, let's use the `crontab` editor.

{% highlight bash %}
$ crontab -e
{% endhighlight %}

This opens a text file with a list of commands meant to be run at specified times. In this first example, we're going to add a job which prints "Hello World" to a log file in every 2 minutes. Add the following line to your crontab file and save it.

<div class="highlight"><pre>
*/2 * * * * php /home/user/Desktop/hw.php >> /home/user/Desktop/hw.log
</pre></div>

A cron job is laid out as follows: minute (0-59), hour (0-23, 0 = midnight), day (1-31), month (1-12), weekday (0-6, 0 = Sunday), command. To run a script more than one time per time unit you can use the syntax described above. This runs the command every 2 minutes. For more information about cron jobs, check out [this tutorial](http://net.tutsplus.com/tutorials/other/scheduling-tasks-with-cron-jobs).

The contents of `hw.php` could simply look like this:

{% highlight php %}
<?php
  echo "Hello World\n";
?>
{% endhighlight %}

Now in every 2 minutes, the log file is updated with a new line. Based on this simple example, we can build a more sophisticated solution which actually does something useful for our web application. For example, the cron job could pull data from an external webservice for caching purposes.

## Additional tips

Depending on your setup, you might run into permission issues. To avoid that, create the cron job under your webserver's user. Apache is usually running under `www-data`.

{% highlight bash %}
$ sudo -u www-data crontab -e
{% endhighlight %}

Include your web application's autoloader into your PHP cron job script. This way you have access to all of your classes.

{% highlight php %}
<?php
  require_once '/path/to/app/autoloader.php';
  ...
?>
{% endhighlight %}

If you are using [Composer](http://getcomposer.org/) in your application, you could make use of [Composer hooks](http://getcomposer.org/doc/articles/scripts.md) and automate the process of adding a cron job. Finally, it is a good practice to implement some form of logging for your background job. You can redirect the standard output to a file as shown previously, or you could use a separate logging framework. In addition, setting up a monitoring environment for cron jobs does not hurt as well.

## End of part 1

This was a simple example on how to use cron jobs for background processing for a PHP application. In the [next part](http://blog.indrek.io/articles/background-jobs-with-php-part-2/ "Part 2 of background jobs in PHP"), we'll look at how to build a queue system with [Resque](https://github.com/resque/resque) and [Redis](http://redis.io).
