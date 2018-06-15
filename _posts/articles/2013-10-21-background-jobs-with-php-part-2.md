---
layout: post
title: "Background jobs with PHP part 2"
excerpt: "In this post, as promised, we're going to implement a queue of jobs with Resque and Redis"
modified: 2013-10-21 23:37:47 +0300
categories: articles
tags: [background, job, php, php-resque, phpredis, redis, resque, tutorial]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

Previously, in [part 1]({{site.url}}/articles/background-jobs-with-php-part-1 "Part 1"), we took a look at how to implement a basic background job with PHP using [cron](http://en.wikipedia.org/wiki/Cron "Cron Wikipedia page"). In this post, as promised, we're going to implement a queue of jobs with [Resque](https://github.com/resque/resque "Resque Github page") and [Redis](http://redis.io/ "Redis homepage").

Redis is an open source, BSD licensed, advanced key-value store. It is often referred to as a data structure server since keys can contain strings, hashes, lists, sets and sorted sets.

## Dependencies

Following software packages are used in this tutorial:

* Redis - key-value store
* Predis - library for accessing Redis key-value store
* php-resque - PHP port of Resque

## Installation

Redis can be downloaded at [http://redis.io/download](http://redis.io/download "Redis download link"). If you're running some flavour of Linux, then you might find Redis in it's package repositories as well. Predis can be acquired at the project's [Github page](https://github.com/nrk/predis "Predis Github page"). It is also available on [Packagist](https://packagist.org/packages/predis/predis "Predis packagist page") for an easy installation using [Composer](http://getcomposer.org/ "Composer homepage"). Finally, php-resque can be found on [Github](https://github.com/chrisboulton/php-resque "php-resque Github page") too. Composer users can find it on [Packagist](https://packagist.org/packages/chrisboulton/php-resque "php-resque on Packagist").

## General overview

We are going to create background workers which are going to process jobs. Jobs are pushed and held in a queue. Workers pull jobs from the queue and execute them. In essence, there are 4 parts to the system:

* Queues - jobs are held here in order
* Jobs - a unit of work to be processed
* Pushers - a process that  pushes  jobs into queues. This process can be anything, a request from the web server or even a worker that creates additional jobs from its previous input.

## Job

Job is an order for the worker to execute a certain task. To create a job with php-resque, you need to create a new class which implements a `perform()`  method. I would create a new interface which defines the `perform()` method and let concrete job classes implement the Job interface.

{% highlight php startinline=true %}
/**
 * Defines a php-resque job
 */
interface Job
{

  /**
   * Executes the job
   */
  public function perform();

}
{% endhighlight %}

{% highlight php startinline=true %}
class HelloJob implements Job
{
  public function perform()
  {
    echo "Hello world";
  }
}
{% endhighlight %}

Of course in a real world application, this job would do something more important and probably be more complex as well. Now that we have a simple job, let's create some workers.

## Workers

To create a new worker, you need to provide some environment variables. Firstly, the QUEUE variable defines which queue the worker is going to poll.


{% highlight bash %}
QUEUE=myQueue php resque.php
{% endhighlight %}

This creates a worker which polls the "myQueue" queue (providing a QUEUE is mandatory). It does this in every 5 seconds. To change the interval, you need to provide the INTERVAL environment variable.

{% highlight bash %}
QUEUE=myQueue INTERVAL=10 php resque.php
{% endhighlight %}

This creates a worker which polls the "myQueue" queue. It does this in every 10 seconds.

Additional variables that might be helpful:

* APP_INCLUDE - path to a file. It can be an autoloader or a file with lot's of includes.
* COUNT - number of workers to create
* REDIS_BACKEND - address and port of your Redis server. Default value is localhost:6379.
* VERBOSE - value 1 will enable debugging messages

If you run:

{% highlight bash %}
QUEUE=myQueue INTERVAL=10 php resque.php
{% endhighlight %}

you should see some output about the created worker, it's PID (process id) and the queue it polls.

{% highlight bash %}
[notice] [15:32:44 2013-10-17] Starting worker my-laptop:17720:myQueue
[info] [15:32:44 2013-10-17] Checking default for jobs
[info] [15:32:44 2013-10-17] Sleeping for 10
{% endhighlight %}

## Running a worker in the background

Currently, if you close your commandline, the worker will be killed as well. To run the command  as a daemon, add an ampersand (&) at the end of the command.

{% highlight bash %}
QUEUE=myQueue INTERVAL=10 php resque.php &
{% endhighlight %}

After running the command, your terminal window is free and you can enter additional commands. But now you can not see the output. Let's log the output to a seperate file.

{% highlight bash %}
QUEUE=myQueue INTERVAL=10 php resque.php >> /path/to/log/workers.log &
{% endhighlight %}

Additionally we could redirect *stderr* to *stdout* with:

{% highlight bash %}
QUEUE=myQueue INTERVAL=10 php resque.php >> /path/to/log/workers.log 2>&1 &
{% endhighlight %}

If the current user is logged out, then the daemon will stop as well. Let's add the following

{% highlight bash %}
nohup QUEUE=myQueue INTERVAL=10 php resque.php >> /path/to/log/workers.log 2>&1 &
{% endhighlight %}

Now the daemon will run even if the user has logged out. In the previous part, i mentioned that in some situations the background job has to run as the same user as the web server. To achieve that we can use the following command:

{% highlight bash %}
nohup sudo -u www-data QUEUE=myQueue INTERVAL=10 php resque.php >> /path/to/log/workers.log 2>&1 &
{% endhighlight %}

Let's recap. This command creates a worker which polls the myQueue queue in every 10 seconds. It's output (*stderr* and *stdout*) is appended to a *workers.log* file. The worker runs with the same privileges as the web server and it runs as a daemon. The daemon is not killed if the user logs out.

## Adding a job to a queue (pushing)

{% highlight php startinline=true %}
Resque::enqueue("default", 'application\models\jobs\TestJob', array("arg1" => "myArg1");
{% endhighlight %}

The first argument is the name of the queue. The second argument is a job class and finally you can provide arguments to your job class. To access the arguments array in the job class, use `$this->args`. If your job class has a namespace then make sure you provide the class' fully qualified name. When this line is executed and you have workers running, you should see "Hello world" printed to your workers.log file.

## Stopping/Killing a worker

First and foremost, to view all the workers currently running, run:

{% highlight bash %}
ps -u | grep resque
{% endhighlight %}

This lists all `resque` processes currently running. To stop a worker, run:

{% highlight bash %}
kill -3 PID
{% endhighlight %}

PID is a process identifier. This is the number in the second column when running `ps -u`. The `-3` flag stops the process when the work has finished. To kill a process, use the `-9` flag.

{% highlight bash %}
kill -9 PID
{% endhighlight %}

## Final thoughts

This was a general overview on how to implement job queues with Redis and Resque. For more information on usage, visit php-resque [Github page](https://github.com/chrisboulton/php-resque "php-resque Github page"). As it can be seen, manual work has to be done to create workers. It would be much more convenient if they were initiated automatically. A bash script is helpful. If you are using Composer, you could use hooks to initiate all the required workers. In addition, the stopping and killing of workers could be automated as well.

During this 2 part series, only a few PHP background processing solutions were covered. But this doesn't mean there aren't any other options available. Do some research and find a solution that works for your specific problem.
