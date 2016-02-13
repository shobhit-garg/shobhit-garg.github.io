---
layout: post
title:  "nginx and reverse proxy" 
date:   2016-02-14
categories: blog
tags: [server,nginx,web]
author: shobhit_garg
share: true
comments: true
excerpt:
---


##What is reverse proxy?

A reverse proxy is a type of proxy server that retrieves resources on behalf of a client from one or more servers.

Let me explain this with an example. When you access google.com through your browser , this request goes to one particular google server mapped with ip address corresponding to google.com. But as we all know only one server can't handle all the requests itself so what it does is it forwards all the requests to other servers but you are unaware of that as you are only communicating with google.com main server.This server which you are communicating with is working as a reverse proxy server as it is retrieving resources from other servers by passing requests to them.

![Diagram]({{ site.url }}/assets/reverse_proxy.gif)

 

 There are multiple uses of reverse proxy server like:

* Can act as a firewall to filter malicious requests.
* Load balancer to distribute the load to different servers.
* Handle SSL of secured websites and may more.


 ##nginx
 
 `nginx` is a web server like Apache. It can act as a reverse proxy server,load balacer and HTTP cache.

 __Installing nginx:__

 On different platforms you can install nginx using different ways. I have tried with OS X and Ubuntu.For other platforms you can search on google or read the [official documentation][nginx-install].

 _On OS X:_
   
  You can install it using package manager `brew`.If you don't have brew in your machine please install it first. 

 `brew install nginx`


 nginx stores the configuration in `nginx.conf` file. If you install nginx using brew default location of this file is /usr/local/etc/nginx/nginx.conf . If you don't find this file here please find it using command `find / -name nginx.conf` .

 _On Ubuntu:_

 You can install it using package management tool `apt-get` .

 `apt-get update`
 `apt-get install -y nginx`
 
 In my case the conf file is located at /etc/nginx/nginx.conf . Again if you don't find it here you can use `find` command to find it's location.

 
 Installing nginx using these methods automatically triggers it.Go to localhost:80 to check nginx has started successfully or not. Now stop nginx using command `sudo nginx -s stop`. For making nginx worked as reverse proxy, you have to edit the nginx.conf file and start nginx again using command `nginx`. 


__nginx.conf__

{% highlight ruby %}

worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    server {
        listen       80;
        server_name  localhost;

		location / {
			root   html;
			index  index.html index.htm;
		}

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }


    }

    include servers/*;

}

{% endhighlight %}

For http request there is http block in conf file. In http block main block is `server` block.
In server block you can see the directives `listen` and `server_name` . nginx first tests the IP address and port of the request against the listen directive of the server blocks. First it checks the port no of request against listen directive value then it checks the host name of request against server_name.As there can be multiple server blocks so first which server block handle the request gets decided using listen and server_name directive. Then comes the role of `location` block inside server block which matches the request url with it's value.

In the given example there is only one server block which is listening to port 80. So if a request comes with url localhost:80 , this server block would handle this request.As you see in this location block, localhost:80 would result in opening a index.html file.

__Setting up reverse proxy:__


 Suppose you are hosting www.example.com and you want to forward the request of www.example.com to port 9000 and request of www.example.com/admin/ to port 9002. For this you have to mention the respective url in `proxy_pass` directive within location block.  As all the request will be passing through nginx server so it would work as a reverse proxy server.


 {% highlight ruby %}

 http
 {

 	.......
 	.......
 	server {

        listen       80;
        server_name  example.com;

		location / {
			proxy_pass http://127.0.0.1:9000/;
		}

		location /admin/ {
			proxy_pass http://127.0.0.1:9002/;
		}

		location /blog/ {
			proxy_pass http://geekdirt.com/;
		}

        ........
        ........

    } 

 }

 {% endhighlight %}

 For more information on it read the official documentation of [nginx reverse proxy][nginx-reverse-proxy].

 [nginx-install]: http://nginx.org/en/docs/install.html
 [nginx-reverse-proxy]: https://www.nginx.com/resources/admin-guide/reverse-proxy/
