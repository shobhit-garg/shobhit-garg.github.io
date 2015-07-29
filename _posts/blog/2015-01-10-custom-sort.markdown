---
layout: post
title:  "Custom Sorting in Ruby"
date:   2015-01-10
categories: blog
tags: [rubyonrails]
author: shobhit_garg
share: true
comments: true
excerpt:
---

We can sort the array of integers in ruby easily,just by using the sort function.But what if we want to __sort array of arrays or array of objects__ etc.
For this we need some thing where we can provide our sorting conditions.Ruby provide this functionality using __sort__ and __sort_by__ functions.

Lets take a example of array of arrays:
{% highlight ruby %}
arr = [[1,4,3],[2,1,7],[3,2,8]]
{% endhighlight %}

#Using sort:

Now you want to sort this array in the __ascending order of the middle element of the inner arrays__.All you need to do is:

{% highlight ruby %}
arr.sort! do |x,y| 
	#if this result is 1 means x should come later relative to y
	#if this result is -1 means x should come earlier relative to y
	#if this result is 0 means both are same so position doesn't matter
	x[1] <=> y[1] #this return 0 or -1 or 1
end

Output : [[2, 1, 7], [3, 2, 8], [1, 4, 3]] 
{% endhighlight %}


Here x,y are the elements of arrays like [[1,4,3]](a element of array) and using x[1] and y[1] we are comparing the middle elements.Now lets look at opeartor <=> :

{% highlight ruby %}
2 <=> 5 # return -1 because 2 is smaller than 5
5 <=> 2 # return 1 because 5 is larger than 2
2 <=> 2 # return 0
{% endhighlight %}

Now if you don't want to use <=> operator then you can do this like:

{% highlight ruby %}
arr.sort! do |x,y|
	if x[1] > y[1]
		1
	elsif x[1] < y[1]
	   -1
	else
	  0
	end
end

#-------------or----------------

arr.sort! do |x,y|
	x[1] - y[1]
end

{% endhighlight %}

#Using sort_by

Here you basically focus on the part of element using which you want to sort the whole array.
{% highlight ruby %}
arr.sort_by!{|x| x[1]}
{% endhighlight %}
