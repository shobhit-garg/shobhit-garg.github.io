---
layout: post
title:  "Blocks,Lambda and Procs in Ruby"
date:   2015-07-25
categories: blog
tags: [rubyonrails]
author: shobhit_garg
share: true
comments: true
excerpt: 
---



Block in a piece of code enclosed with {} or do...end.

Example:

{% highlight ruby %}

test {puts "You are in a block"}

{% endhighlight %}



#__How to use block?__

#Case 1: 

__Passing a block to a function not as a function argument : Use 'yield' to access__

if a block is passed to a function , function can run it's code using yield. If at the time of calling the function, block is not passed then accessing the block would throw exception.So the better way is to check before using that block is passed or not in the function and to check this you can use __block_given?__ .Check the below examples for better understanding.


{% highlight ruby %}

#function
def speak
  puts yield
end

#function
def speak_safe
	puts yield if block_given?
end

1.
#calling this function by passing block {"hello"}
speak { "Hello" }
#Hello

2.
#calling this function by passing block do "hello" end
speak do "hello" end 
#Hello

3.
#calling this function by passing block {"hello"}
speak_safe { "Hello" }
#Hello

4.
#calling this function without passing block 
speak
#LocalJumpError: no block given (yield)

5.
#calling this function without passing block 
speak_safe 
#nil

{% endhighlight %}




#Case 2: 

__Passing a block to a function as a function argument : Using 'call' to access__


Prepend '&' in the last argument name if block is passed in it.In case both * and & are present in the argument list, & should come later.Block attached to this method is converted to a Proc object and gets assigned to that last argument.
You can use yield here too.


{% highlight ruby %}

#function
def speak(&block)
	puts block.call
end

#calling this function by passing block {"hello"}
speak() {"hello"} #same as speak hello
#hello

#calling this function without passing block 
speak() #or speak
#NoMethodError: undefined method `call' for nil:NilClass

{% endhighlight %}


#Blocks which expect values

Arguments can be passed to blocks.For this within the block, you list the names of the arguments to receive the parameters between vertical bars (|..|).

{% highlight ruby %}

#function
def multiply(&block)
	puts block.call(4,5)
	puts yield(10,20)
	puts block.class
end

#calling function by passing block
test() {|i,j| i*j}

#20 
#200
#Proc (Read the complete article to get an idea about Proc) 

{% endhighlight %}

__Explanation__ : Block {|i,j| i*j} is passed to function and using block.call and yield , block runs.


#__Procs__

Procs in Ruby are first-class objects, since they can be created during runtime, stored in data structures, passed as arguments to other functions and returned as the value of other functions.

{% highlight ruby %}

p = Proc.new{|i| i*2}
p.call(2)
#4

{% endhighlight %}

If you put & in front of a Proc instance in the argument position, that will be interpreted as a block.Using this logic block is passed to a function and later get called.

{% highlight ruby %}

def check(&t)
	t.call #t is a Proc object
	puts t.class
end

p = Proc.new{puts "hello"}
check(&p) #passing block not proc

#hello
#Proc

{% endhighlight %}

#__lambda__

lambda is used to create Proc object. It's same as Proc except some minor differences.


{% highlight ruby %}

a = lambda {|k| puts k}
#<Proc:0x007faf706258f0@(irb):70 (lambda)> 
#It's creating a proc object.Please note.
a.class
#Proc 
a.call(2)
#2

#function
def multiply(block)
	puts block.call(4,5)
end

l = lambda {|a,b| a*b}
#<Proc:0x007faf74933340@(irb):75 (lambda)> 
multiply(l)
#20

p = Proc.new {|a,b| a*b}
#<Proc:0x007faf7464c320@(irb):78> 
multiply(b)
#20

{% endhighlight %}


__Symbol#to_proc__

Many times you have seen in code statements like:


{% highlight ruby %}

Order.all.map(&:id)
#[1,2,3,4....800] (all order ids)

#Same as
Order.all.map{|o| o.id}
#[1,2,3,4....800] (all order ids)

{% endhighlight %}

But have you ever wondered how come this works? Given is the reason:


The & operator is combined with :method_name (&:method_name), which is not a Proc instance, so implicit class cast applies Symbol#to_proc to it, which gives ->x{x.method_name}

{% highlight ruby %}
#reverse is a String function to reverse
my_proc = :reverse.to_proc 
 #<Proc:0x007faf74959f40>
my_proc.call("abc")
#"cba"

{% endhighlight %}

For more information please visit this [blog][ror_block_blog]


[ror_block_blog]:    http://eli.thegreenplace.net/2006/04/18/understanding-ruby-blocks-procs-and-methods/





