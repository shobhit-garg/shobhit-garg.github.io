---
layout: post
title:  "Deep and Shallow copy in Rails"
date:   2015-02-04
categories: blog
tags: [rubyonrails]
author: shobhit_garg
share: true
comments: true
excerpt:
---

__Shallow Copy:__ They don't copy the objects that might be referenced within the copied object.In rails dup and clone are used for this.

__Deep Copy:__ All the objects are copied.In rails deep_dup and deep_clone are used for this.


__Shallow Copy:__

{% highlight ruby %}
a = {1 => [2,3,4]}
b = a.dup
a[1]<< 5
a
# => {1=>[2, 3, 4, 5]} 
b
# => {1=>[2, 3, 4, 5]} 

a[1].object_id
# => 70283040856600 
b[1].object_id
# => 70283040856600

{% endhighlight %}
__Deep Copy:__

{% highlight ruby %}

a1= {1 => [2,3,4]} 
b1 = a1.deep_dup
a1[1]<< 5
b1
# => {1=>[2, 3, 4]} 
a1
# => {1=>[2, 3, 4, 5]} 

a1[1].object_id
# => 70283036017040 
b1[1].object_id
# => 70283036034520 

{% endhighlight %}


__clone does two things that dup doesn't:__

__1.__ Copy the singleton class of the copied object

{% highlight ruby %}

#shallow copy
a = Object.new
def a.foo; :foo end
p a.foo
# => :foo
b = a.dup
p b.foo
# => undefined method `foo' for #<Object:0x007f8bc395ff00> (NoMethodError) vs clone:

{% endhighlight %}

{% highlight ruby %}

#deep copy
a = Object.new
def a.foo; :foo end
p a.foo
# => :foo
b = a.clone
p b.foo
# => :foo

{% endhighlight %}

__2.__ Maintain the frozen status of the copied object

{% highlight ruby %}

a = Object.new
a.freeze
p a.frozen?
# => true

b = a.dup
p b.frozen?
# => false

c = a.clone
p c.frozen?
# => true

{% endhighlight %}
