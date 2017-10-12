---
layout: post
title:  "Hadoop-3: How Map-Reduce works (Partitioning, Shuffling, Combiner)?"
date:   2017-10-13
categories: blog
tags: [docker,server,hadoop,bigdata,mapreduce,yarn]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---

In the first post of Hadoop series [Introduction of Hadoop and running a map-reduce program][hadoop-part-1] , i explained the basics of Map-Reduce. In this post i am explaining its different components like Partitioning, Shuffle, Combiner, Merging, Sorting first and then how it works. 

__Partitioning:__

Map task returns output in <key,value> form. For many of the outputs key can be same. For example if map output is <1,10> , <1,15> , <1,20> , <2, 13> , <2,6> , <4,8> , <4,20> etc. we can see that there are 3 different keys which are 1, 2 and 4. In map-reduce number of reduce tasks are fixed and each reduce task should handle all the data related to one key. That means map output like <1,10>, <1,15>, <1,20> should be handled by same reduce tasks. It is not possible that <1,10> is handled by one reduce task and <1,15> is handled by another reduce task as key which is 1 is same. Partitioning is to club the data which should go to the same reducer based on keys. One common partitioning approach is hash based partitioning in which partition number is calculated using hash(key) % number_of_reducers. 

If number of reducers are 2 and hash function is:

{% highlight ruby %}

#This function would return the data passes in it
function hash(data){
	#this is just for example purpose
	return data;
}

{% endhighlight %}

then for key 1:

partition number = hash(1) % 2 =  1;

for key 2:

partition number = hash(2) % 2 =  0;

for key 4:

partition number = hash(4) % 2 =  0;

That means output for key 1 would go in partition_1 and output for key 2 and 4 would go in partition_0.  

__Shuffle:__

In the final output of map task there can be multiple partitions and these partitions should go to different reduce task. Shuffling is basically transferring map output partitions to the corresponding reduce tasks. Map task notified application master about completion of map task and application master notifies corresponding reducer to copy the map output into reduce machine. 

__Combiner:__

Reducing the data on map node from map output so that reduce task can be operated on less data. Like map output in some stage is <1,10>, <1,15>, <1,20>, <2,5>, <2,60> and the purpose of map-reduce job is to find the maximum value corresponding to each key. In combiner you can reduce this data to <1,20> , <2,60> as 20 and 60 are maximum value for key 1 and key 2 respectively. 

__Sorting:__

It is just sorting the data based on keys.

__Merging:__

This happens on reducer side. Reducer can get data from multiple map tasks and through merging it merges the data of different map tasks in one single unit, maintaining the sorting order.

---------------------------------------------------
Map-Reduce in detail:

![Diagram]({{ site.url }}/assets/map_reduce_detail.png)
<br><br>

__Map:__

It has an in memory `circular buffer` and it writes directly to buffer. When buffer reaches the threshold value (default 80%) it starts writing to `spill file` through a separate `background thread`. Map task can still write it's output to buffer while background thread is running. Before writing to spill file, map output that is key-value pairs are partitioned and sorted. If a combiner is there, it also gets run that reduces the output size. 

After the task, all the spill files are sorted again and a single file is created which is partitioned and sorted. Means this final output file has multiple partition. After map task in completed for a node its output is sent to reducer. That's why you see reduce task more than 0% while map is less than <100%.

When you have a map-only task, there is not shuffling at all, which means that mappers will write the final output directly to the HDFS.


__Reduce:__ 

After getting data from map task it merges the data into one single unit during the `merge phase`. Instead of merging directly all files into one, it uses the concept to `merge factor` , purpose of which is to minimize the amount of data written to disk. Then during reduce phase, the reduce function is invoked for each key in the sorted output. The output of this phase is written directly to the output file system, typically HDFS.

Useful Links:

[Introduction and working of Yarn][hadoop-part-2]

[hadoop-part-1]: {{site.url}}/blog/introduction-to-hadoop/
[hadoop-part-2]: {{site.url}}/blog/introduction-and-working-of-yarn/

