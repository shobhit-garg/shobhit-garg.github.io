---
layout: post
title:  "Javascript & Node : COMMONJS and AMD"
date:   2016-05-01
categories: blog
tags: [web,node,server,javascript]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---

Javascript has not been my strong area but currently i am working on `nodejs`. While working on that i got to know that nodejs follows `commonjs` module approach.So the question is what is this commonjs? But before starting that let me explain a few things :-


How most developer write javascript code:

{% highlight javascript linenos %}
<script>
	var x = 10;
</script>

{% endhighlight %}

or they write the same type of code in a file and include that file in html. Here they are polluting the global space. Because variable x is global now and any function might change it. Also it will be in the memory all the time.



To get rid of this many javascript developers come with different moduler approaches and famous ones are AMD(Asynchronous Module Definition ) and COMMONJS. 

A typical `AMD module` looks like it:

{% highlight javascript linenos %}

define(['dep1', 'dep2'], function (dep1, dep2) {

    //Define the module value by returning a value.
    return function () {};
});

{% endhighlight %}

dep1 and dep2 are other modules which would be loaded before calling this. It follows asynchronous approach that means dep1 and dep2 would be loaded asynchronously.


A typical `COMMONJS` module looks like this:

{% highlight javascript linenos %}
var dep1 = require("dep1");

function add(a,b)
{
	return a+b;
}

exports.add = add;

{% endhighlight %}

It includes require and exports statements. You must have seen this in nodejs if you have worked on it.It follows synchronous approach.



But the thing here is you can't run these modules directly through browser without an external library. That is where module loader like `requirejs` comes into picture. AMD modules can be loaded and run using requirejs directly. But to run commonjs modules,the most common approach is to put the commonjs code is this block:

{% highlight javascript linenos %}
define(function(require, exports, module) {
    //Put traditional CommonJS module content here
});
{% endhighlight %}

In nodejs you can directly run commonjs modules without any external library.

External Links:
 
[requirejs][requirejs]

[Using commonjs with requirejs][commonjs_requirejs]


[requirejs]: http://requirejs.org/docs/api.html
[commonjs_requirejs]: http://requirejs.org/docs/commonjs.html
