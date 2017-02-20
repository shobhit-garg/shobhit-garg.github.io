---
layout: post
title:  "Google Analytics Walkthrough"
date:   2015-10-10
categories: blog
tags: [analytics,google-analytics]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---

Before starting this please go through [Basics of Analytics][basics-of-analytics] to understand the basics of analytics. 


## Reports

There are bascially 4-5 types of reports in google analytics:


__Real Time:__ As the name infers, it display information like how many users are currently visiting the site, what are their location etc. 

![Real Time Report]({{ site.url }}/assets/real-time.png)
*Real Time Report*

__Audience:__ Who your users are? It tells about your users demographics , locations, browser/mobile they are using etc. Information like how many of them are first time and how many of them are returning is also the part of this.

![Audience Report]({{ site.url }}/assets/audience.png)
*Audience Report*

__Acquisition:__ How users arrive at your web site? Like users are coming from which sources/medium, on which pages they are landing , what are their flow. If they are coming through some adwords or campaign then information about this too.

![Acquisition Report]({{ site.url }}/assets/acquisition.png)
*Acquisition Report*

__Behavior:__ How users interact with your site or app? Like total number of pages seen per session, the order in which screens are viewed,  how long a typical session lasts.	

![Behavior Report]({{ site.url }}/assets/behavior.png)
*Behavior Report*

__Conversions:__ It's for goals and ecommerce about which i would be discussing in some other post.



## Account, Property , Views and Segments

You can create an `account` on google analytics.There can be multiple `property` in an account. Each property has it's own `tracking code` and there is no relation between two properties. Generally people create two property one for web and other for app to keep the analytics separate.

In each property there can be multiple `views`. Views are nothing but filtered results.Default view is 'All website data' for web. You can create other views like in which you can filter requests from some particular host, particular ISP , particular country etc. So like for a property we can have views like All Website Data , US Data, Europe Data , Asia data. So that you can analyze them separetly. Data starts populating for a view as you create it, means when you create a view it doesn't fetch data from other views.

![Account, Property and Views]({{ site.url }}/assets/views.jpg)
*Account, Property and Views*

A Segment is a subset of your Analytics data. For example, of your entire set of users, one Segment might be users from a particular country or city. Another Segment might be users who purchase a particular line of products or who visit a specific part of your site. Maximum four segments can be there in a view at a time.

![Segments]({{ site.url }}/assets/segment.jpg)
*Segments*


## Goals


Goals is what you want to achieve. Like user should buy a product from your website, a user is interacting with your site more than 30 min, a user is watching a particular video on your website etc. You can create your custom goal. You can also give a value to goal. Like if a user is visitng site for 5 minutes it's a goal and it's worth 1 dollor. Using this you can anaylze your business in terms of revenue like if you are using web advertising than you can compare the money you are spending on advertisement and the value you are getting. Be careful while adding goal value to an ecommerce transaction.

There are different types of custom goals you can create through google analytics:

1.Destination : Like make Thank You page a goal.

2.Duration

3.Pages or Screen per Visit

4.Event

![Duration based goal]({{ site.url }}/assets/goal.png)
*Duration based goal*

## How google analytics works?

On loading the page google creates a cookie on browser which stores the unique user id. This cookie name is `_ga` , you can check this in your browser cookie section. The value of _ga is something like GA1.3.1807876512.1433250537. Here 1807876512 is the client id and 1433250537 is the timestamp. Combination of both create a unique global user id.
When you create a property in google analytics , google provide you a `tracking id` which is something like UA-65776519-1. So using this tracking id and global user id , google sends data to it's servers. Using tracking id google identify the data is for which website or app and using global user id google identifies the user.





[basics-of-analytics]: {{site.url}}/blog/basics-of-analytics/
