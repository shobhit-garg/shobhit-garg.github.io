---
layout: post
title:  "Redis and Resque"
date:   2015-03-11
categories: blog
tags: [rubyonrails,cache]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---


__REDIS__

 It is basically a key-value cache and store. The name Redis means REmote DIctionary Server.


Key should be a string like "user:1000:followers" and value can be of many data structures like hash, list, set etc. Unlike __memcache__ in which key and value can be strings only.

Redis does not use SQL to inspect its data, instead having its own command set to read and process the keys. It provides a command-line interface, redis-cli, to interactively view and manipulate the dataset.

Redis Lists

{% highlight ruby %}

redis> rpush mylist "hello, redis"  # <= Adds the value to the right side of the list/queue
# mylist is key here
(integer) 1

redis> type mylist                  # <= Returns the datatype of the value of this key
list

redis> lrange mylist 0 10           # <= Returns a elements 0 through 10 from the list/queue
1) "hello, redis"

{% endhighlight %}


Redis Hashes


{% highlight ruby %}

> hmset user:1000 username antirez birthyear 1977 verified 1
# user:1000 is key here
OK
> hget user:1000 username
"antirez"
> hget user:1000 birthyear
"1977"
> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"

{% endhighlight %}


__RESQUE__

Resque is a Redis-backed library for creating background jobs, placing those jobs on multiple queues, and processing them later.
It leans on the Redis __list datatype__, with __each queue name as a key, and a list data type as the value__.

Each job in Resque is a hash serialized as a JSON string (remember data structures can not be nested in Redis) of the format:

{"class":"MyModel", "args":[123, "arg1", "arg2", ...]}

As mentioned resque uses redis list data structure. So many values of the JSON type mentioned above can be put in a single queue.

In class we define a perform method which gets called when resque worker runs the job. As jobs in resque contains class name and parameters so before running resque idenfities the class and call the perform method based on parameters.

To enqueue a job in Resque you need to do :

{% highlight ruby %}

Resque.enqueue(ClassName,Parameters)

{% endhighlight %}

This creates a job in JSON format on redis server. Jobs are en-queued (the Redis RPUSH command to push onto the right side of the list) on the list, and workers de-queue a job (LPOP to pop off the left side of the list) to process it. As these operations are atomic, queuers and workers do not have to worry about locking and synchronizing access. Data structures are not nested in Redis, and each element of the list (or set, hash, etc.) must be a string.

In class you can put @queue = queue_name which decides in which queue job should be put.


A Resque worker will then try to process the job.
You can start up a worker with

{% highlight ruby %}

$ bin/resque work

{% endhighlight %}

This will basically loop over and over, polling for jobs and doing the work. You can have workers work on a specific queue with the --queue option:

{% highlight ruby %}

$ bin/resque work --queues=high,low
$ bin/resque work --queue=high

{% endhighlight %}

You can provide the redis location to Resque through file resque.rb located in config/initializers.







