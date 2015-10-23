---
layout: post
title:  "How to iterate HashMap in Java"
date:   2015-06-23
categories: blog
tags: [java]
author: shobhit_garg
share: true
comments: true
excerpt:
---

Suppose this is the HashMap you are defining:

{% highlight ruby %}
HashMap<K,V> h = new HashMap<>();
{% endhighlight %}


In HashMap.java, there is this decleration.

{% highlight ruby %}
transient Set<Map.Entry<K,V>> entrySet;
{% endhighlight %}

This Set stored all the Entry class objects.Entry class objects are the objects which stores key-value pair in HashMap.Entry class has fields [K key,V value,Map.Entry<K,V> next] etc. This diagram will give you a idea about how java implement HashMap and store key-values:


![HashMap Implementation in Java]({{ site.url }}/assets/hashmap.jpg)



__Code to Iterate:__

{% highlight ruby %}
Set<Map.Entry<K,V>> set = h.entrySet();
#entrySet() function returns a Set which contains all the Map.Entry<K,V> objects present in HashMap.
#So you need to iterate over a Set now.
Iterator<Map.Entry<K,V>> it = set.iterator();

         while(it.hasNext())
         {
             Map.Entry<K,V> pair = it.next();
             V value = pair.getValue();
             K key = pair.getKey();
             #you can use value and key according to your need
         }

{% endhighlight %}

__Usecase Example:__

Suppose you are storing ArrayList as a Value(V) in HashMap.Now you need to sort all the ArrayList's stored in HashMap.You can do this via iterating over HasMap and get the Object of ArrayList then you can sort it.
