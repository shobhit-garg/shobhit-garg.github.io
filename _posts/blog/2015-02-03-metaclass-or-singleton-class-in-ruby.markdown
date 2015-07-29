---
layout: post
title:  "Metaclass/Singleton class in Ruby"
date:   2015-02-03
categories: blog
tags: [rubyonrails]
author: shobhit_garg
share: true
comments: true
excerpt:
---

In Ruby a class itself is an instance of the Ruby class.
AnyClassName.class  => Class

Objects can also have methods that are independent of the parent class definition. They are called singleton methods and are stored on the metaclass of the object.When you declare a singleton method on an object, Ruby automatically creates a class to hold just that method. This class is called the 'metaclass' of the object.Whenever you send a message to the object, it first looks to see whether the method exists in its metaclass. If it is not there, it gets propagated to the actual class of the object and if it is not found there, the message traverses the inheritance hierarchy.

Ruby classes themselves have their own singleton classes since classes are objects as well. Basically these type of methods belong to a single object rather than to an entire class and other objects.The class << idiom is simply Ruby's syntax for accessing the scope of an object's singleton(meta) class.

{% highlight ruby %}

Object.new.singleton_class  #=> #<Class:#<Object:0xb7ce1e24>>
#Object.new creates an instance of class Object.Which has a meta class(singleton class).So <Class:#<Object:0xb7ce1e24>> is signifying a Class(metaclass) of an instance of Object.

String.singleton_class      #=> #<Class:String> 
#metaclass of Class String

{% endhighlight %}

{% highlight ruby %}

Obj = Anyclass.new
def Obj.anymethod
"Hi am a singlton method available only for firstObj"
end

{% endhighlight %}

We are defining methods particular to an object.It would be defined in it's metaclass.
For getting all singleton method for an object use:

Obj.singleton_methods

{% highlight ruby %}

[1].class
 => Array 
 #The actual class of Array object

 [1].singleton_class
 => #<Class:#<Array:0x007feb40cd3f88>> 
 #Basically the meta class of Array Object

 {% endhighlight %}



__Use of << :__

For defining metaclasses or scoping methods for a particular metaclass.






{% highlight ruby %}

class Person
  def testm
    puts self  #=> instance of Person(#<Person:0x007ff7a350d258>)
    class << self #=> defining Object's metaclass becuase it's inside object method
      puts self  #=> class(metaclass) of instance of Person(#<Class:#<Person:0x007ff7a35219b0>>)
    end
  end
end

{% endhighlight %}


{% highlight ruby %}
class Person
  # currently self is class Person so here we are defining it's metaclass so every method defined in it would be a method of Person's metaclass.
  # Which can directly be accessed through Person.method_name
  class << self 
  end
end
{% endhighlight %}
