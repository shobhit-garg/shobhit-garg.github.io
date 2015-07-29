---
layout: post
title:  "Different type of INDEXES in database / Primary Key - Unique Key"
date:   2015-06-25
categories: blog
tags: [database]
author: shobhit_garg
share: true
comments: true
excerpt:
---


I suppose you might be aware of the concept of Database Indexing.If you are not, please read about how Indexes works.
In this post my main focus is to let you know the differences between different types of Indexes.


__Normal Indexes__ :

 These are the most basic indexes, and have no restraints such as uniqueness or Null value.

{% highlight ruby %}
CREATE TABLE people (
	peopleid SMALLINT NOT NULL AUTO_INCREMENT,
	age SMALLINT ,
	birth_year SMALLINT ,
	PRIMARY KEY (peopleid),
	INDEX normal (age,birth_year) #normal index
);
#table created
{% endhighlight %}

{% highlight ruby %}

insert into people values ('1','1','NULL');
insert into people values ('2','1','NULL');
insert into people values ('3','1','1990');
#all successful

{% endhighlight %}


__Unique Indexes__ : 

Unique indexes are the same as Normal indexes with one difference: all values of the indexed column(s) must only occur once.
Unique indexes allow null values in it's columns if they allow null values.You can name it according to your choice .In the example i am using name my_uniq_index .

{% highlight ruby %}

CREATE TABLE people (
peopleid SMALLINT NOT NULL AUTO_INCREMENT,
age SMALLINT ,
birth_year SMALLINT ,
PRIMARY KEY (peopleid),
UNIQUE my_uniq_index (age,birth_year)
);
#table created
{% endhighlight %}

{% highlight ruby %}

insert into people values ('1','1','NULL'); 
#successful

insert into people values ('2','1','NULL');
#Duplicate entry '1-0' for key 'my_uniq_index'

insert into people values ('3','1','1');
#successful

insert into people values ('3','1','1');
#Duplicate entry '1-1' for key 'my_uniq_index'

{% endhighlight %}



__Primary Keys/Indexes__ : 

Primary keys are unique indexes that must be named PRIMARY.You may only have one primary key per table.
If you are using AUTO_INCREMENT then that column must be defined as PRIMARY key.Unlike Unique key it doesn't allow NULL values.

{% highlight ruby %}

CREATE TABLE people (
peopleid SMALLINT NOT NULL AUTO_INCREMENT,
age SMALLINT
)
#Incorrect table definition; there can be only one auto column and it must be defined as a key.

{% endhighlight %}


{% highlight ruby %}

CREATE TABLE people (
peopleid SMALLINT NOT NULL AUTO_INCREMENT,
age SMALLINT,
PRIMARY KEY (peopleid)
);
#table created
#correct way of doing this if you are using autoincrement.

{% endhighlight %}



{% highlight ruby %}

CREATE TABLE people (
peopleid SMALLINT,
PRIMARY KEY (peopleid)
);
#table created

insert into people values(NULL);
#Column 'peopleid' cannot be null

insert into people values('1');
#successful

insert into people values('1');
#Duplicate entry '1' for key 'PRIMARY'

{% endhighlight %}







