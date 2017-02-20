---
layout: post
title:  "Cache in Rails and Memcached"
date:   2015-03-20
categories: blog
tags: [rubyonrails,cache]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---


__Memcached__

Memcached uses a client–server architecture. The servers maintain a key–value associative array; the clients populate this array and query it. Keys are up to 250 bytes long and values can be at most 1 megabyte in size.


* Most of memcache functionality (add, get, set, flush etc) are o(1). This means they are constant time functions.

* Before you start a memcache-daemon, you have to tell it how much memory it should use. It will allocate that memory right from the start.
If memory gets full it uses LRU(least recently used) method to preserve keys and delete keys.


* It uses Consistent Hashing so that all the memcached-servers get the equal load of cached data. You can read more about consistent hashing [here][memcached]. Servers are dumb- they don't know about each other.


__Caching in Rails:__

To enable caching in development environment:

{% highlight ruby %}

#Inside config/environments/development.rb
config.action_controller.perform_caching = true

{% endhighlight %}

On production it' true by default.


There are three types of caching in Rails: __Page Caching,Action Caching,Fragment Caching__ . You can read more about caching on [this][cache] . Here the caching has been explained in a pretty good way with examples.


Caching is not part of Rails 4.0 core.So you have to use these gem in your rails application.

{% highlight ruby %}

gem 'actionpack-page_caching' #for page caching.
gem 'actionpack-action_caching' #for action caching.

{% endhighlight %}


Rails provides different stores for the cached data created by action and fragment caches.Page caches are always stored on disk.

By default, Rails 3 will initialize a FileStore that you can reference through Rails.cache. This cache is used internally by Rails to cache classes, pages, actions, and fragments. If you want to change which cache store is used, you can configure it in your application.rb.

Example: Using a MemoryStore cache with a max size of 512 megabytes

{% highlight ruby %}
config.cache_store = [:memory_store, {:size => 536870912}] # user can use any store
{% endhighlight %}

__Implementing a Cache Store:__


ActiveSupport::Cache::Store < Object is an abstract cache store class. You have to implement this class for creating your own cache store. There are multiple cache store implementations, each having its own additional features. See the classes under the ActiveSupport::Cache module, e.g. ActiveSupport::Cache::MemCacheStore. MemCacheStore is currently the most popular cache store for large production websites.MemcachedStore uses memcached gem as backend to connect to Memcached server.


[memcached]: https://www.adayinthelifeof.nl/2011/02/06/memcache-internals/ 
[cache]: http://www.codelearn.org/blog/rails-cache-with-examples









