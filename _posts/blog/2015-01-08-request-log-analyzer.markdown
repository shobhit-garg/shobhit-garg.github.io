---
layout: post
title:  "Analyzing Logs:Request Log Analyser"
date:   2015-01-08
categories: blog
tags: [rubyonrails,logs]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---

We anaylze production logs time to time.There are many gems/tools available for it.But i found [request-log-analyzer][rla] a good tool to use.To install this gem use 
{% highlight ruby %}
gem install request-log-analyzer
{% endhighlight %}

__General Use:__

If you want to analyse completly(without any custom option) just run this command:
{% highlight ruby %}
request-log-analyzer log/development.log #path of log file
{% endhighlight %}

This generates a detailed report having these informations:

1.Request distribution per hour

2.Most requested actions

3.HTTP methods(get,post etc.) usage

4.Duration a request is taking with thier mean and deviation

5.Partial/View rendering time.

6.Routing errors etc.


__Custom Formats:__

Now say, You log the data when you make an api call from your app.Your log format is like this:

Calling API [api_name] at [time]

Now you want to check how many times a particular api is being called.Request Log Anaylser provides custom formats to anaylse this type of logs.

Create a file named [filename].rb and paste this code into our file.
{% highlight ruby %}
 class MyFormat < RequestLogAnalyzer::FileFormat::Rails3
  # Define line types
  line_definition :my_line_type do |line|
    line.regexp = /Calling API (.)*/ #You should modify this according to your log format
    line.captures << { :name => :memcached, :type => :string }
    puts line
  end

    # define the summary report
    report(:append) do |analyze|
      analyze.frequency :memcached, :title => "Count of API calls", :line_type => :my_line_type
    end

  end
{% endhighlight %}

Now to use this custom format use command line:
{% highlight ruby %}
request-log-analyzer log/development.log --format my_format.rb #assuming my_format.rb is the name of file i have created.
{% endhighlight %}

This will generate the full report and append the report related to our custom format at last.


[rla]:   https://github.com/wvanbergen/request-log-analyzer
