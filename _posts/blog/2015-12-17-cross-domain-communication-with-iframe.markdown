---
layout: post
title:  "Cross Domain communication with iframe"
date:   2015-12-19
categories: blog
tags: [web]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---


People generally load iframe of different domain/origin in their webpage. Like may sites have the iframe of youtube loaded in it. Sometimes you need to communicate with these iframes but due to cross origin security reasons you are not able to do it.

Consider a case when you host a video player on your server and you are embedding this video player in your webpage which is in different domain. Now you want to display the lyrics in your webpage as you start playing the video in your iframe. This time you own the iframe too, so there should be a way to communicate with your webpage.


The way we use for this is called window message passing. To explain this clearly i have hosted two websites, address of first one is http://localhost:8888 and second one is http://localhost:9000. In the page of http://localhost:8888, i am embedding iframe of http://localhost:9000 . So here the parent is http://localhost:8888 and the child is http://localhost:9000 .


![Diagram]({{ site.url }}/assets/iframe.png)


`index.php` of http://localhost:8888

{% highlight ruby %}
<body>

<iframe id = "iframe1" name = "iframe1" src = "http://localhost:9000"  height = "700px"  width = "500px">
</iframe>

</body>

{% endhighlight %}

## Sending message from Parent to Child

__Sending message from parent__

{% highlight ruby %}

 #javascript
 document.getElementById("iframe1").contentWindow.postMessage("hi_from_parent_8888","http://localhost:9000");

{% endhighlight %}




 __Receiving message in child__

{% highlight ruby %}

function listener(event){
 
  console.log(event.origin);
  console.log(event.data);
  #any action you can write here
}

if (window.addEventListener){
  addEventListener("message", listener, false)
} else {
  attachEvent("onmessage", listener)
}

{% endhighlight %}

On listening event.origin would return http://localhost:8888 & event.data would return test hi_from_parent_8888. So you can apply a check at the time of receving too to apply action only on messages coming from valid domains.


##Sending message from Child to Parent 

__Sending message from child__

{% highlight ruby %}

parent.postMessage("hi_from_parent_9000","http://localhost:8888");

{% endhighlight %}

__Receiving message in parent__

{% highlight ruby %}

 function listener(event){
 
  console.log(event.origin);
  console.log(event.data);
  #any action
}

if (window.addEventListener){
  addEventListener("message", listener, false)
} else {
  attachEvent("onmessage", listener)
}

{% endhighlight %}