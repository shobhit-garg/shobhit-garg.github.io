---
layout: post
title:  "Flash messages in Rails"
date:   2015-05-31
categories: blog
tags: [rubyonrails]
author: shobhit_garg
share: true
comments: true
excerpt:
---

The flash provides a way to pass temporary primitive-types (String, Array, Hash) between actions. Anything you place in the flash will be __exposed to the very next action and then cleared out__.


{% highlight ruby %}

class PostsController < ActionController::Base
  def create
    # save post
    flash[:notice] = "Post successfully created"
    redirect_to @post #redireting to show
  end

  def show
    # doesn't need to assign the flash notice to the template, that's done automatically
    #here you can access flash[:notice] too
  end
end


show.html.erb
  <% if flash[:notice] %>
    <div class="notice"><%= flash[:notice] %></div>
  <% end %>

{% endhighlight %}

Since the notice and alert keys are a common idiom, convenience accessors are available:

{% highlight ruby %}

flash.alert = "You must be logged in" #or flash[:alert]
flash.notice = "Post successfully created" #or flash[:notice]

{% endhighlight %}

You can use any idiom like:

{% highlight ruby %}

flash[:flag] = true
puts flash.flag 
#undefined method `abcd' 
#because flash doesn't provide accessor for this unlike notice and alert

{% endhighlight %}


__Persist flash values against multiple actions:__

Now suppose you have three actions say action1, action2 , action3.And you are calling one to another:

{% highlight ruby %}
def action1
 	flash.notice = 'check'
 	redirect_to :action2
end


def action2
	puts flash.notice
	#'check'
	redirect_to :action3
end


def action3
	puts flash.notice
	#nil
end

#Because flash persist values only for the next action. It will clean the flash object on redirecting to action3.
#keep is used to persist value in flash for the next action.

{% endhighlight %}

__Use of keep:__

{% highlight ruby %}
def action1
 	flash.notice = 'check'
 	redirect_to :action2
end


def action2
	puts flash.notice
	#'check'
	flash.keep(:notice) 
	#keep the notice value for the next action
	redirect_to :action3
end


def action3
	puts flash.notice
	#print 'check'
	#now if you redirect to some action4 and want to persist the value of flash you again have to use keep.
end

{% endhighlight %}


__Persisting values just for currect action:__

Can't access these values in next action.

{% highlight ruby %}
flash.now.notice = "Good luck now!"
#or use flash.now[:notice] 
{% endhighlight %}




