---
layout: post
title:  "Hadoop-1: Introduction of Hadoop and running a map-reduce program"
date:   2017-04-09
categories: blog
tags: [docker,server,hadoop,bigdata,mapreduce]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---



In this post i am giving a brief overview of Hadoop. Extensive overview like how it works and what makes it highly distributive and fault tolerant will be given in later posts. Target of this post is to make you comfortable in running your first Hadoop job.

## Hadoop Basics

In simple terms, Hadoop is a framework to process large datasets in distributed environment. 

Hadoop includes four basic modules:

__1. Hadoop common:__

All the java libraries and utilities required by Hadoop modules.

__2. Hadoop YARN:__

Framework for job scheduling.

__3. HDFS:__

The Hadoop Distributed File System (HDFS) is the primary storage system used by Hadoop applications. It means it can store the large data on different nodes connected through network. It is highly fault-tolerant and scalable. HDFS stores the metadata in NameNode and actual content in DataNodes. There can be many DataNodes across the network, all of which store file content in blocks of standard size. Namenode monitors the replication of blocks and keep the metadata updated.

For accessing files/directory in HDFS , command `bin/hdfs dfs` as a prefix is used.

List all the files of input folder :

{% highlight javascript %}

bin/hdfs dfs -ls input/*

{% endhighlight%}

Remove all the files of output folder:

{% highlight javascript %}

bin/hdfs dfs -rm rf  output/*

{% endhighlight%}

![Diagram]({{ site.url }}/assets/hdfsarchitecture.gif)
<u><br>HDFS Architecture (Image source:Hadoop Offical Documentation) </u>

__4. MapReduce:__

Hadoop MapReduce is a software framework for writing application which process vast amount of data in parallel on large cluster (thousand of nodes). It basically consists of two parts:

Map : It takes a set of data and converts it into intermediate set of data which must be of <key,value> pair form.

Reduce: It takes input tuples from map and combine those tuples into smaller set of tuples.

Example: Say you want to count the occurrence of words given in a file:

{% highlight javascript %}

Hello
World
Bye
World
Hello
Hadoop
Goodbye
Hadoop

{% endhighlight %}

You put this data into two Map jobs, output of Map jobs would be like:

The first map emits:

{% highlight javascript %}

< Hello, 1> 
< World, 1> 
< Bye, 1> 
< World, 1> 

{% endhighlight %}

The second map emits:

{% highlight javascript %}

< Hello, 1> 
< Hadoop, 1> 
< Goodbye, 1> 
< Hadoop, 1> 

{% endhighlight %}

And when data is passed to Reduce job, it gets converted into:

{% highlight javascript %}

< Bye, 1> 
< Goodbye, 1> 
< Hadoop, 2> 
< Hello, 2> 
< World, 2> 

{% endhighlight %}

(It's a simple example to explain MapReduce jobs. You can also use Combine after Map jobs which first reduce the data at Map level then further reduction is at Reduce level.)


-----------------------------------------------------

## Setting up Hadoop using docker

In the previous tutorial [Introduction of Docker and running it on Mac][docker-intro] i have mentioned how to set up docker. Please go through this post and set up docker on your machine. Then run this command to download and run hadoop container:

{% highlight javascript %}

docker run -it sequenceiq/hadoop-docker:2.7.1 /etc/bootstrap.sh -bash

{% endhighlight %}

(I am using version 2.7.1 . You can check the latest version on [hadoop-docker][sequenceiq-hadoop-docker])


To check hadoop is working try to run a Grep job using:

{% highlight javascript %}

cd $HADOOP_PREFIX

bin/hadoop jar /usr/local/hadoop-2.7.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar grep input output 'dfs[a-z.]+'

{% endhighlight %}

( Here we are running a example jar which has many map-reduce jobs. You can find the source code of these jobs from [hadoop-examples][hadoop-examples]. After the jar you have to specify which job you want to run and here we are running `grep` job which needs three params which are input directory, output directory and grep pattern respectively.)


Once you run this program you can find the output in output directory which you can see through command:

{% highlight javascript %}

bin/hdfs dfs -ls output/*

//It returns
output/_SUCCESS
output/part-r-00000

{% endhighlight %}

_SUCCESS is a blank file denoting the success of map reduce job.
part-r-00000 contains the output. You can view the file using command:

{% highlight javascript %}

bin/hdfs dfs -cat output/part-r-00000

{% endhighlight %}



## Running your MapReduce program in Hadoop:


__WordCount.java__

{% highlight java %}

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}

{% endhighlight %}

Please put this file on location $HADOOP_PREFIX/mapreducejob/WordCount.java .


__Compiling the program__

Different ways to compile a map reduce program:

<u>1. Using the minimal need libraries</u>

{% highlight javascript %}

cd $HADOOP_PREFIX

javac -cp share/hadoop/common/hadoop-common-2.7.1.jar:share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.7.1.jar mapreducejob/WordCount.java 

{% endhighlight %}
 


<u>2.Hadoop minimilisitc way</u>

{% highlight javascript %}

cd $HADOOP_PREFIX
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar

bin/hadoop com.sun.tools.javac.Main mapreducejob/WordCount.java 

{% endhighlight %}

<u>3. Using the hadoop actual classpaths</u>

{% highlight javascript %}
cd $HADOOP_PREFIX

export HADOOP_CLASSPATH=$(bin/hadoop classpath)
javac -classpath ${HADOOP_CLASSPATH} mapreducejob/WordCount.java 

{% endhighlight %}

__Creating Jar__

{% highlight javascript %}

cd mapreducejob

jar cf wc.jar WordCount*.class

cd ..


/*
  For creating a jar we are changing the directory. 
  You can use simple jar cf wc.jar mapreducejob/WordCount*.class 
  but for that you have to define package mapreducejob; 
  in WordCount.java and while running you have to use 
  mapreducejob.WordCount as classname.
*/

{% endhighlight %}

__Running Job__

{% highlight javascript %}

bin/hadoop jar mapreducejob/wc.jar WordCount input/yarn-site.xml output

//Here 'input/yarn-site.xml' file is the name of file for which we
//are counting the occurrences  of words. I am using this file as 
//this already exists in hadoop file system. You can create your 
//own file too in HDFS and give that path. 'output' is the output directory.

{% endhighlight %}


If you get this exception while running a job,

Exception in thread "main" org.apache.hadoop.mapred.FileAlreadyExistsException: Output directory ,

Please clear the output directory first using:

{% highlight javascript %}

bin/hdfs dfs -rm rf  output/*
bin/hdfs dfs -rmdir  output 

{% endhighlight %}



You can view the HDFS file system using the web view but for that you have to expose the docker port 50070 . For that run hadoop image using:  

{% highlight javascript %}

docker run -it -p 50070:50070 sequenceiq/hadoop-docker:2.7.1 /etc/bootstrap.sh -bash

{% endhighlight %}

Now you can access hdfs file system through web on 127.0.0.1:50070. 

![Diagram]({{ site.url }}/assets/HDFSUI.png)
<u><br><br> Web View HDFS </u>


__Important Links:__

[Hadoop-2: Introduction of YARN and how it works?][hadoop-part-2]


[sequenceiq-hadoop-docker]: https://github.com/sequenceiq/hadoop-docker
[docker-intro]: {{site.url}}/blog/docker-and-running-it-on-mac/
[hadoop-examples]: https://github.com/facebookarchive/hadoop-20/tree/master/src/examples/org/apache/hadoop/examples
[hadoop-part-2]: {{site.url}}/blog/introduction-and-working-of-yarn/





