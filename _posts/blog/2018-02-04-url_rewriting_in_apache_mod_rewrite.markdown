---
layout: post
title:  "Url rewriting in Apache using mod_rewrite : RewriteCond & RewriteRule"
date:   2018-02-04
categories: blog
tags: [apache,web,server,mod_rewrite]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---

mode_rewrite is an `Apache module` which uses a `rules based writing engine` to rewrite requested url on fly like a url to a file system path, redirect an url and to invoke an internal proxy fetch . For example if user is requesting an url 'www.example.com?course=ml&class=regression', mode_rewrite can change the url to 'www.example.com/course/ml/class/regression' on server and serve the request. It can validate a url based on few conditions and decide to process the request further or not.

There are two main directive of this module: `RewriteCond` & `RewriteRule` . RewriteRule is used to rewrite the url as the name signifies if all the conditions defined in RewriteCond are matching. One or more RewriteCond can precede a RewriteRule directive. If we talk about traditional programming RewriteCond works just like 'If' condition where you can use conditions like AND, OR, >=, == , ! etc. 

RewriteCond C1       
RewriteCond C2      
RewriteRule R     

Which means rewrite rule R would work only if both the conditions C1 `AND` C2 get satisfied. If you want to use `OR` condition you have to explicitly mention it. For example:

RewriteCond C1 [OR]    
RewriteCond C2     
RewriteRule R      

In this case rule R would work only if condition C1 gets satisfied `OR` condition C2 gets satisfied. 

There can be multiple RewriteRules and they get executed in top to bottom order. For example:

RewriteCond C1        
RewriteCond C2         
RewriteRule R1         

RewriteCond C3         
RewriteRule R2        

RewriteRule R3     

Here there are three RewriteRules R1, R2 and R3. R1 has conditions C1 AND C2 , R2 has condition C3 and R3 has no condition. So for a url first rule R1 will get applied if C1 and C2 get satisfied then R2 if C3 gets satisfied then R3. In top to bottom order if one RewriteRule doesn't get executed then the control goes to other, for example if R1 is not applied because of rule is not matching for the requested url or may be because conditions C1 and C2 are not matching then control would go to R2 and so on. Each RewriteRule gets a chance to process itself in the same order if you are not using some special `flag L` etc. which we talk about later.

------------------------------
__Components of Url:__     


This image would help in explaining different components of a url.

![Diagram]({{ site.url }}/assets/url_structure.png)


__Use of Regular Expressions__

Although there is lot to write on url rewriting in apache but i will try to keep the things simple, once you get the basics you can easily implement more complex rules by reading the [offical documentation][mod_rewrite]. RewriteCond and RewriteRule both use `groups` concept of `regular expressions`. Those who are familiar with regular expression might know about it, for others i am explaning regular expression parenthetical groups here. 

Inside a regular expression, expression with parenthesis are called groups and you can get the matched values of these groups directly. Say you want to get the path(index.html) of a url(www.abc.com/index.html), use a regular expression which is ^www.abc.com\/(.\*)$ , here (.\*) is a group because it contains a parenthesis. For example purposes i am using javascript to run regular expressions, concept in every language is same.

/^www.abc.com\/(.\*)$/.exec("www.abc.com/index.html")      
(path index.html)

Output : [ "www.abc.com/index.html" , "index.html" ]

After running this you get an array , at 0th location of array full matched result is there and on index 1, matched group url path 'index.html' is there. In case of multiple groups resultant array size increases by no. of groups.


Now want to check the url of type www.abc.com/[x]/[y] and we want both subpaths [x] and [y] after matching. So we use a regular expression /^www\.abc\.com\/(.\*)\/(.\*)$/ which contains two groups (two parenthetical expressions) and we run it against this url "www.abc.com/sports/shoes"

/^www.abc.com\/(.\*)\/(.\*)$/.exec("www.abc.com/sports/shoes")        
(path sports/shoes)           
Output :  [ "www.abc.com/sports/shoes" , "sports" , "shoes" ]               

Here you can see at 0th index full matched url is there, at index 1 first grouped matched url subpath 'sports' is there and at index 2 second grouped matched url subpath 'shoes' is there.

-------------------------

Now coming to the point, we use the same concept in RewriteRule and RewriteCond.
In RewriteRule $0 points to the whole matched path, $1 points to the first matched group, $2 points to the second matched group and so on.  If you are using regular expressions in RewriteCond then you can refer these groups in RewriteRule using % instead of $ like %1 , %2 . Examples given below would make this clear.


__Variables__

There are few `variables` which you can use to refer some standard data, few of them are :

%{HTTP_HOST} -> Host name of url. Like www.example.com or example.com

%{HTTP_REFERER} -> Like from abc.com , you are clicking on a link to example.com then abc.com is a referrer.

%{QUERY_STRING} -> Url after question mark(?). Like in www.abc.com?id=5, id=5 is query string.

%{REQUEST_URI} -> Path component of request. Like in www.abc.com/examples/ml, examples/ml is request uri. 

__Flags__

There are few `flags` which you can mention with RewriteCond and RewriteRule, few of them are:

L - last to parse       
F - url forbidden      
N - begin rewriting process again      
R  - Redirect       
NC - case insensitive      

One thing to keep in mind for RewriteRule is that path part of url is passed to RewriteRule not the complete url. Before RewriteRule you have to mention `RewriteEngine On` once to get rewriting url working. Examples would make it clear.


__<u>Examples:</u>__

__Redirecting old url to new url:__

RewriteEngine On        
RewriteCond %{HTTP_HOST} ^www.oldurl.com$ [OR]       
RewriteCond %{HTTP_HOST} ^oldurl.com$      
RewriteRule ^$  http://newurl.com [L,R=301]       

Input Url: www.oldurl.com        
Output Url: http://newurl.com         
(with redirection)

Explanation: Input url is www.oldurl.com , %{HTTP_HOST} is www.oldurl.com so first RewriteCond is matching. As there is OR  between conditions so RewriteCond part is returning true. Now come to RewriteRule, as i mentioned earlier path element of url is passed to RewriteRule and here path is empty in www.oldurl.com. 
So empty string is matching with ^$ and output url is http://newurl.com. Flag R is used to mention a redirect with status code 301(Permanent moved) and flag L is used to mention this is the last RewriteRule to execute if there are more RewriteRules. 


__Changing path from oldpath to newpath:__

RewriteEngine On        
RewriteRule ^oldpath$  newpath         

Input Url: www.example.com/oldpath        
Output Url: www.example.com/newpath        

Explanation: Path element is input url is 'oldpath' which is getting passed to RewriteRule and it is matching with ^oldpath$ so oldpath is getting replaced with newpath as per RewriteRule. We have not specified any redirect flag here, so client would always think that request is getting served from www.example.com/oldpath, but server is actually serving it from www.example.com/newpath by rewriting the url.


__Changing query string into path:__        
(Use of regular expression grouping)

RewriteEngine On     
RewriteCond %{QUERY_STRING} ^id=([0-9]\*)$        
RewriteRule ^(.\*)$ $0/id/%1?         

Input Url: www.example.com/course?id=5         
Output Url: www.example.com/course/id/5          

Explanation: Path element of input url is 'course' and query string is 'id=5' . In the RewriteCond we are matching query string through regular expression ^id=([0-9]\*)$ so the output of this would be [ 'id=5' , '5' ] means 1st element of output would represent the number '5' and to use it in RewriteRule we have to use %1. %1 would contain '5'. Now come to RewriteRule part, path 'course' is passed to regular expression ^(.\*)$ , so output of execution this would be [ 'course' , 'course'] as there is one group(check the parenthesis). So in RewriteRule both $0 and $1 will represent 'course'. So $0/id/%1 means course/id/5 .
Important thing here is if group is coming from RewriteRule then you have to use $ , if it is coming from RewriteCond you have to use %. There is ? at last in RewriteRule which is used to indicate black query string in final output.


__Redirecting invalid urls to home page:__

RewriteEngine On     
RewriteRule !.(js\|gif\|jpg\|png\|css\|eot\|svg\|ttf\|woff\|woff2\|map)$ index.html

Input Url: www.example.com/home.js         
Output Url: www.example.com/home.js

Input Url: www.example.com/random.hack        
Outpur Url: www.example.com/index.html


Explanation: ! is negation operator. In first one, path in url is 'home.js' which is matching with regular expression .(js\|....)$ and negation(!) of that is false so url is not rewritten and same url is returned. In second one path in url is 'random.hack' which is not matching with regular expression .(js\|....)$ and negation(!) of that is true so path is getting changed to 'index.html' so final url is www.example.com/index.html .





__Important Links:__

You can try your RewriteRule on [https://htaccess.mwl.be/][test_rewrite_rule].

For more depth you can read this [article][mode_rewrite_depth] .


[mod_rewrite]: http://httpd.apache.org/docs/current/mod/mod_rewrite.html
[test_rewrite_rule]: https://htaccess.mwl.be/
[mode_rewrite_depth]: https://code.tutsplus.com/tutorials/an-in-depth-guide-to-mod_rewrite-for-apache--net-6708
