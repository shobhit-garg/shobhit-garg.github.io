---
layout: post
title:  "Introduction of Docker and running it on Mac"
date:   2017-02-16
categories: blog
tags: [docker,server,osx]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---

You might have been familiar with `Virtual Machine` . A VM can be run on any operating system using the hypervisor such as VMWare, Virtualbox etc. VM contains a whole operating system and applications installed by users. Size of VMs is large and processing is slow as VM relies on it's own OS kernel not on host system's OS kernel. Also VM's OS uses a good amount of resources provided to VM which slows down the application.
The advantage of VM is that you can install a number of libraries and applications there, set system variables and configure the environment and save it. Later you can run this VM on any machine using the hypervisor without worrying about setting the dependencies and libraries again.


![Diagram]({{ site.url }}/assets/dockervsvm.jpg)
<u><br>Virtual Machine and Docker (Image source:Docker Offical Documentation) </u>

You can achieve the same thing through Docker in more enhanced and effective way. Docker just uses a layer of `docker-engine` over OS layer to run applications through it, so it's a light weight process compare to running VMs. Running a docker image also takes a lot lesser time as compare to VM. There are two main things in docker:

__Docker Image:__
An image is a filesystem and parameters to use at runtime. You can club all your libraries,applications and other dependencies in one docker image. This image can be used on any operating system with docker.

__Docker Container:__
A container is a running instance of image. Using an image you can create multiple number of containers.

So docker gives a significant performance boost and reduces the size of the application. It is used widely in deploying applications.


## Installing docker on Mac:

There are no. of resources which can guide you how to install docker on your operation system.Please google it. As i work on mac, i am mentioning here how to install docker on mac.

There are two way to use docker on mac:

__1.Using docker-machine__

Kind of virtual machine using which you can run docker. Using this approach first you have to go on docker-machine terminal then only you can use docker commands.This approach is for older mac systems.Please visit [docker-machine][docker-machine] to get more information.

__2.Using Docker for Mac__

This is the recommended approach. Docker for Mac is a tool using which you can run docker directly on Mac. You can download the executable(dmg) from [here][docker-for-mac] and install it. After installing, you can find it in Applications folder and can run it like other appliatons.

## Important commands and starting a Container

Hopefully docker would be running in your system by now.Here is the list of few important commands:

`docker images`

It displays all the images present locally.

`docker ps`

It displays all the running containers.

`docker run [OPTIONS] IMAGE_NAME`

It starts the container of that image. If image is not present locally, it fetches the image from global repository.

`docker stop CONTAINER_ID`

It stops the docker container given container id which you can get by running docker ps .

<u>Running nginx using docker</u>

docker run -d nginx

(-d is for running in background)

It will start the container of nginx. Now run docker ps, it will show the container information and the ports on which it is running. 

{% highlight javascript %}
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
ea93a955c65c        nginx               "nginx -g 'daemon ..."   7 seconds ago       Up 5 seconds        80/tcp, 443/tcp     sleepy_stonebraker
{% endhighlight  %}

As you see you can access nginx on port 80 and 443.But to access these ports from localhost we have to map these ports to our machine ports. For now stop this container using 'docker stop CONTAINER_ID' , CONTAINER_ID in this case is ea93a955c65c.

Starting nginx container again by connecting port 80 of container to port 8080 of localhost:

docker run  -p 8080:80 -d nginx

Now you can access the nginx on http://localhost:8080 . 

[docker-for-mac]: https://download.docker.com/mac/stable/Docker.dmg
[docker-machine]: https://docs.docker.com/machine/overview/


