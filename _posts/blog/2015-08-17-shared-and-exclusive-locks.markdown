---
layout: post
title:  "Shared and Exclusive locks in Mysql/UPDATE FOR & LOCK IN SHARE MODE query"
date:   2015-08-17
categories: blog
tags: [transaction,database]
author: shobhit_garg
comments: true
share: true
excerpt: ""
---

I am assuming you have gone through [transactions and locks][transactions-and-locks] before starting this.

Before starting this , let's discuss an example simple database transaction:

__Case:Supppose there are two sessions which are trying to add a discount of 1000 of order value is greater than 4500.__


{% highlight ruby %}
#Here mysql1 is indicating session1 and mysql2 is indicating session2.

mysql1 > start transaction
mysql1 > select order_value from orders where id = 21548;
#5000


mysql2 > start transaction
mysql2 > select order_value from orders where id = 21548;
#5000

#Now session1 has read that value is 5000 which is greater than 4500
#so it tries to add discount

#(in if block)
mysql1 > update orders set order_value = order_value - 1000 , discount = discount + 1000 where id = 21548


#And session2 has read the same value which is 5000 and greater than 4500
#so it tries to add discount too

#(in if block)
mysql2 > update orders set order_value = order_value - 1000 , discount = discount + 1000 where id = 21548


mysql1 > commit

mysql2 > commit

{% endhighlight %}


After completion of this if you try to check the state of database, you will get `dirty data` .

{% highlight ruby %}
mysql1 > select order_value,discount from orders where id = 21548;
#3000,2000
{% endhighlight %}

Only a discount of 1000 should be given but in this case you end up giving 2000 discount. That means only __transaction is not the solution of every problem__. Now the question arrives is how to make this thread safe ? The answer is locking.

-------------------------------------------------------------------------------------------------------------------------------


__Row level locking:__ 

Shared (S) locks: It permits a transaction to read a row. If transaction of one session has got this lock no transaction of other session can apply X lock on the same row. We can also say that no transaction of some other session can update that row.

Exclusive (X) locks: It permits a transaction to update or delete a row. If transaction of one session has got this lock , no transaction of some other session can apply S or X lock on the same row.
In a simple way we can say that other session can't update that row but it can only read that row without asking for any lock.


Official definitions:

If transaction T1 holds a shared (S) lock on row r, then requests from some distinct transaction T2 for a lock on row r are handled as follows:

A request by T2 for an S lock can be granted immediately. As a result, both T1 and T2 hold an S lock on r.

A request by T2 for an X lock cannot be granted immediately.


If a transaction T1 holds an exclusive (X) lock on row r, a request from some distinct transaction T2 for a lock of either type on r cannot be granted immediately. Instead, transaction T2 has to wait for transaction T1 to release its lock on row r.



__Table level locking:__

`Intention Locks`

InnoDB supports multiple granularity locking which permits coexistence of row locks and locks on entire tables.  The idea behind intention locks is for a transaction to indicate which type of lock (shared or exclusive) it will require later for a row in that table. There are two types of intention locks used in InnoDB:

Intention shared (IS): Allows requestor to lock rows in S or IS mode.

Intention exclusive (IX): Allows requestor to explicitly lock rows in X, S, IX or IS mode.




__FOR UPDATE & LOCK IN SHARE MODE in mysql__

select .. from .. where .. FOR UPDATE

select .. from .. where .. LOCK IN SHARE MODE

The whole target of this post is to let you know about these two things. Both of these are used with select query.
FOR UPDATE  locks the entire table in IS mode and the selected rows in S mode.
LOCK IN SHARE MODE locks the entire table in IX mode and selected rows in X mode.

Please check the following example for deep understanding.

{% highlight ruby %}

mysql1 > select * from orders where id = 21548 FOR UPDATE; 
#There is no transaction in this so it immedietly releases the lock.

mysql2 > update orders set order_value = 1000 where id = 21548;
#Works becuase there is no lock applied in connection1.

{% endhighlight %}


{% highlight ruby %}

mysql1 > start transaction;
mysql1 > select * from custom_orders where id = 21548 FOR UPDATE;
#acquired IX lock on table and X lock on row with id 21548



mysql2 > select * from orders where id = 21548; 
#works
mysql2 > update orders set order_value = 1000 where id = 21549; 
#works
mysql2 > select * from orders where id = 21548 FOR UPDATE; 
#waiting for lock which is acquired by connection1 on row with id 21548 

{% endhighlight %}

{% highlight ruby %}

mysql1 > start transaction;
mysql1 > select * from orders where id = 21548 FOR UPDATE;
#acquired IX lock on table and X lock on row with id 21548

mysql2 > start transaction
mysql2 > select * from orders where id = 21548 FOR UPDATE; #waiting
#waiting for lock which is acquired by connection1 on row with id 21548 

{% endhighlight %}

{% highlight ruby %}

mysql1 > start transaction;
mysql1 > select * from orders where id = 21548 LOCK IN SHARE MODE;
#acquired IS lock on table and S lock on row with id 21548 so can't update this row by different connection

mysql2 > update orders set order_value = 1000 where id = 21549;	
#works because row with id 21549 is not locked.
mysql2 > update orders set order_value = 1000 where id = 21548;
#waiting for lock which is acquired by connection1 on row with id 21548 


{% endhighlight %}


{% highlight ruby %}

mysql1 > start transaction;
mysql1 > select * from orders where id = 21548 LOCK IN SHARE MODE;
#acquired IS lock on table and S lock on row with id 21548 so can't update this row by different connection


mysql2 > select * from custom_orders where id = 21548 FOR UPDATE;
#waiting for lock which is acquired by connection1 on row with id 21548 
#This is asking for X lock but can't be given as connection1 has S lock on the same row.

{% endhighlight %}



__MySQL must maintain locks on every row it updates until the transaction commits so it can roll them all back at once if the transaction fails or is cancelled.__

{% highlight ruby %}

mysql1 >  start transaction;
mysql1 > update orders set order_value = 1000 where id = 21548;
#As this is happening in transaction , mysql automatically applies lock on the concerned row.


mysql2 > select * from orders where id = 21548 LOCK IN SHARE MODE;
#waiting for the lock acquired by connection1 on the same row. 

{% endhighlight %}

For any doubts please check this question on [Stack Overflow][is-ix-lock-question] .

[is-ix-lock-question]: http://stackoverflow.com/questions/31880185/how-locks-s-x-is-ix-work-in-mysql-with-queries-like-for-update-lock-in-share-m
[transactions-and-locks]: {{site.url}}/blog/transaction-and-locks-in-mysql/




