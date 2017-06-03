---
layout: post
title:  "Hadoop-2: Introduction of YARN and how it works?"
date:   2017-06-03
categories: blog
tags: [docker,server,hadoop,bigdata,mapreduce,yarn]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---

In the last blog [Introduction of Hadoop and running a map-reduce program][hadoop-part-1] , i explained different components of hadoop, basic working of map reduce programs, how to setup hadoop and run a custom program on it. If you follow that blog you can run a map reduce program and get familiar with the environment a little bit. Before starting this post i recommend to go through the previous post once. The purpose of this post is to go a little deep and describe how YARN works and how it is better from previous Hadoop 1.0 environment. Then we will look into some configuration files using which we can manage hadoop environment better. 


In the last post i mentioned there are four basic modules of hadoop which are:

1. Hadoop common

2. Hadoop Yarn

3. HDFS

4. MapReduce

We have discussed about HDFS and Mapreduce. Here i am explaining architecture and working of YARN in detail.

## YARN (Yet Another Resource Negotiator)

YARN was introduced in Hadoop 2.0. In Hadoop 1.0 a map-reduce job is run through a job tracker and multiple task trackers. Job of job tracker is to monitor the progress of map-reduce job, handle the resource allocation and scheduling etc. As single process is handling all these things, Hadoop 1.0 is not good with scaling. Also it makes Job tracker a single point of failure. In 1.0, you can run only map-reduce jobs with hadoop but with YARN support in 2.0, you can run other jobs like streaming and graph processing. In 1.0 slots are fixed for map and reduce tasks so while map is running you can't use reduce slots for map tasks because of that slots go waste, in 2.0 there is a concept of container, which has resources like memory and cpu-cores and any task can be run in it.

YARN has basically these component: 


__Resource Manager:__ It has two main component: Job Scheduler and Application Manager. Job of scheduler is allocate the resources with the given scheduling method and job of Application Manager is to monitor the progress of submitted application like map-reduce job. It has all the info of available resources. 

__Node Manager:__ For each node there is a node manager running. It maintains the available resources on that particular node and notifies Resource Manager about the available resources when it starts. It launches the containers by providing the needed resources (memory, cpu etc.). These resources are allocated to container by Resource Manager.
It manages the containers during it's lifetime. It sends heartbeat to Resource Manager to let it know that it is alive. In case Resource Manager doesn't receive heartbeat from Node Manager, it marks that node as failure.

__Application Master:__ It carries out the execution of job using different components of YARN. It is spawned under Node Manager under the instructions of Resource Manager . One Application master is launched for each job. For resource allocation it talks to Resource Manager, for launching or stopping a container it talks to Node Manager. It aggregates the status of task from different nodes and notifies the status of job to client as client polls on it. It also sends periodic heartbeat to Resource Manager to make sure Resource manager can launch a new Application Master in case of failure.


__Container:__ It is started by Node Manager. It consists of resources like memory, cpu core etc. For running a map or reduce task, Application Master asks Resource Manager for resources using which a container can be run. 


## Steps involved in running a job using YARN:

![Diagram]({{ site.url }}/assets/yarn.png)
<u><br>Anatomy of a YARN Application Run (from "Hadoop: The Definitive Guide" by Tom White):</u>

(Steps numbers given in diagram are different so don't get confused.)

1. User submits jobs to Job Client present on client node.

2. Job client asks for an application id from Resource Manager.

3. Job which consists of jar files, class files and other required files is copied to hdfs file system under directory of name application id so that job can be copied to nodes where it can be run. 

4. Job is submitted to Resource Manager.

5. Resource Manager contacts Node Manager to launch a new container and run Application Master in it.

6. Application Master checks the splits (usually blocks of datanode of hdfs) on which job has to runs and create one task per split usually. Only ids are given to all the task in this phase. It checks if all the tasks can be run sequentially on same JVM  on which Application Master is running then it doesn't launch any new containers. This type of job is called `uber job`. 

7. If job is not an uber job, Application Master asks Resource Manager for allocating the resources. Resource manager knows after node manager hdfs blocks and their bandwidth, so it allocate resources considering the data locality so that tasks can be run on same machine on which data blocks are present. 

8. Application manager gets the resources information from Resource Manager and it launches the container through Node Manager. In container the task is executed by the java application whose main class is `YarnChild` . Before running the task it copies all the job resources from hdfs. In most of the cases, job programs which is usually in jar form are copied to machine on which data is present.

9. Task sends progress update to Application master time to time. In case of failure Application master can launch the task on some other container. In case of run time exception JVM reports to Application master and in case of JVM failure Node manager notifies Application Master.




I checked some logs of Client, Resource Manager and Node Manager on submitting a job. These might also give you some idea about the working of YARN.


{% highlight ruby %}

#logs on Client when you submit a job:

17/06/03 05:28:23 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
17/06/03 05:28:25 INFO input.FileInputFormat: Total input paths to process : 31
17/06/03 05:28:25 INFO mapreduce.JobSubmitter: number of splits:31
17/06/03 05:28:25 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1496481984963_0001
17/06/03 05:28:25 INFO impl.YarnClientImpl: Submitted application application_1496481984963_0001
17/06/03 05:28:26 INFO mapreduce.Job: The url to track the job: http://9a7e9cf75fe1:8088/proxy/application_1496481984963_0001/
17/06/03 05:28:26 INFO mapreduce.Job: Running job: job_1496481984963_0001
17/06/03 05:28:34 INFO mapreduce.Job: Job job_1496481984963_0001 running in uber mode : false
17/06/03 05:28:34 INFO mapreduce.Job:  map 0% reduce 0%

......
......


#--------------------------------------------------------

# logs on Resource Manager:

2017-06-03 05:28:25,893 INFO org.apache.hadoop.yarn.server.resourcemanager.ClientRMService: Application with id 1 submitted by user root
2017-06-03 05:28:25,893 INFO org.apache.hadoop.yarn.server.resourcemanager.rmapp.RMAppImpl: Storing application with id application_1496481984963_0001

.....

2017-06-03 05:28:26,861 INFO org.apache.hadoop.yarn.server.resourcemanager.scheduler.SchedulerNode: Assigned container container_1496481984963_0001_01_000001 of capacity <memory:2048, vCores:1> on host 9a7e9cf75fe1:44643, which has 1 containers, <memory:2048, vCores:1> used and <memory:6144, vCores:7> available after allocation
......

2017-06-03 05:28:33,869 INFO org.apache.hadoop.yarn.server.resourcemanager.scheduler.SchedulerNode: Assigned container container_1496481984963_0001_01_000005 of capacity <memory:1024, vCores:1> on host 9a7e9cf75fe1:44643, which has 5 containers, <memory:6144, vCores:5> used and <memory:2048, vCores:3> available after allocation

......

2017-06-03 05:28:34,291 INFO org.apache.hadoop.yarn.server.resourcemanager.rmcontainer.RMContainerImpl: container_1496481984963_0001_01_000005 Container Transitioned from ALLOCATED to ACQUIRED

.....

2017-06-03 05:28:50,723 INFO org.apache.hadoop.yarn.server.resourcemanager.scheduler.SchedulerNode: Released container container_1496481984963_0001_01_000005 of capacity <memory:1024, vCores:1> on host 9a7e9cf75fe1:44643, which currently has 5 containers, <memory:6144, vCores:5> used and <memory:2048, vCores:3> available, release resources=true
2017-06-03 05:28:50,723 INFO org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.LeafQueue: default used=<memory:6144, vCores:5> numContainers=5 user=root user-resources=<memory:6144, vCores:5>

......
......


#--------------------------------------------------------
#logs on Node Manager:
2017-06-03 05:28:27,249 INFO org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: Start request for container_1496481984963_0001_01_000001 by user root
....

2017-06-03 05:28:27,314 INFO org.apache.hadoop.yarn.server.nodemanager.containermanager.localizer.LocalizedResource: Resource hdfs://9a7e9cf75fe1:9000/tmp/hadoop-yarn/staging/root/.staging/job_1496481984963_0001/job.jar transitioned from INIT to DOWNLOADING
....
2017-06-03 05:28:50,504 INFO org.apache.hadoop.yarn.server.nodemanager.containermanager.container.ContainerImpl: Container container_1496481984963_0001_01_000005 transitioned from EXITED_WITH_SUCCESS to DONE
.....
2017-06-03 05:28:51,184 INFO org.apache.hadoop.yarn.server.nodemanager.containermanager.monitor.ContainersMonitorImpl: Stopping resource-monitoring for container_1496481984963_0001_01_000005
.......
.......


{% endhighlight %}

--------------------------------------------------------------

## Configuration files in Hadoop

These are some of the configuration files which you can change according to your need. You can find these files in $HADOOP_HOME/etc/hadoop. Some of them are as following:

__hadoop-env.sh__ 

Set environment variables that are used in the scripts to run Hadoop. Some important ones are:

`JAVA_HOME` : Jave home path to avoid any confusion

`HADOOP_HEAPSIZE` : Memory processes like Application Master/ Resource Manager/ Node Manager / Datanode etc. should take.

`HADOOP_LOG_DIR` (By default log directory location is $HADOOP_HOME/logs)

__yarn-env.sh__

Set environment variables that are used in the scripts to run YARN. Overrides settings set by hadoop-env.sh. Some important ones are:

`YARN_RESOURCE_MANAGER_HEAPSIZE` : Memory a RM should take

`YARN_NODEMANAGER_HEAPSIZE` : Memory a Node Manager should take

__mapred-env.sh__

Set environment variable that are used in script to run MapReduce. Overrides settings set by hadoop-env.sh.


__core-site.xml__

Configuration settings for hadoop core such as I/O settings that are common to HDFS, MapReduce and YARN. One of them is:

`fs.defaultFS` : defining name node location of hdfs

__hdfs-site.xml__

Configuration settings for namenodes and datanodes.

__yarn-site.xml__

Configuration settings for YARN daemons, the resource manager, web app proxy server and node managers


Some settings are given by the client in the job configuration, like:

`mapreduce.map.memory.mb` - amount of memory for map containers

`mapreduce.reduce.memory.mb` - amount of memory for reduce containers







You can check the progress of a Job through web interface on `http://127.0.0.1:8088` . You have to expose this port if you are running hadoop through docker. Please check the previous post for this.

![Diagram]({{ site.url }}/assets/hadoop_cluster.png)
<u><br>Hadoop cluster web interface</u>

Important Links:

[Hadoop-1: Introduction of Hadoop and running a map-reduce program][hadoop-part-1]


References:

* Hadoop, The Definitive Guide by Tom White

* [Job Run YARN in hadoop video][yarn-youtube-video]




[hadoop-part-1]: {{site.url}}/blog/introduction-to-hadoop/
[yarn-youtube-video]: https://www.youtube.com/watch?v=oz8TwPgDhc4
