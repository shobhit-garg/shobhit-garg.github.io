---
layout: post
title:  "Transaction and Locks in Mysql"
date:   2015-05-26
categories: blog
tags: [database,transaction,security]
author: shobhit_garg
share: true
comments: true
excerpt:
---

#Transactions

Transactions are required for atomicity.If one part of the transaction fails, the entire transaction fails, and the database state is left unchanged.

* START TRANSACTION or BEGIN start a new transaction

* COMMIT commits the current transaction, making its changes permanent

* ROLLBACK rolls back the current transaction, cancelling its changes

Suppose we have two client(mysql1 and mysql2) session with the mysql server.In db we have an orders table.


Effect of transaction of one session on other session:

{% highlight ruby %}

#session 1
mysql1> select amount from orders where id = 1;
1000

mysql1> START TRANSACTION;
mysql1> update orders set amount = 500 where id = 1;

#session 2
mysql2> select amount from orders where id = 1;
1000

#because database creates a replica of data which is not in commited state.It always return the commited data to each session.As we have not commited the session 1 transaction yet that's why it is showing amount as 1000.

#session 1
mysql1> COMMIT

#session 2
mysql2> select amount from orders where id = 1;
500 # you got it right?

{% endhighlight %}



Both session try to update the different tuples:

{% highlight ruby %}

#session 1
mysql1> select amount from orders where id = 1;
1000

mysql1> START TRANSACTION;
mysql1> update orders set amount = 500 where id = 1;

#session 2
mysql2> select amount from orders where id = 1;
1000

mysql2> select amount from orders where id = 2;
2000

mysql2> update orders set amount = 1500 where id = 2;


#session 1

mysql1> select amount from orders where id = 2;
2000

mysql1> COMMIT

mysql1> select amount from orders where id = 2;
2000

#session 2
mysql2> select amount from orders where id = 1;
500 #because transaction at connection 1 is commited

mysql2> COMMIT

#session 1
mysql1> select amount from orders where id = 2;
1500

{% endhighlight %}



Both session try to update the same tuples:

{% highlight ruby %}

#session 1
mysql1> select amount from orders where id = 1;
1000

mysql1> START TRANSACTION;
mysql1> update orders set amount = 500 where id = 1;

#session 2
mysql2> select amount from orders where id = 1;
1000


mysql2> update orders set amount = 1500 where id = 1;
#(this is now dependend on session 1 transaction becuase you are trying to update the same tuple.
So this session would start waiting.You can't do any operation this time.)


#session 1

mysql1> select amount from orders where id = 1;
500
mysql1> COMMIT
#As you commit the waiting at session 2 side will get over.

mysql1> select amount from orders where id = 1;
500


#session 2
mysql2> select amount from orders where id = 1;
1500
mysql2> COMMIT

#session 1
mysql1> select amount from orders where id = 1;
1500

{% endhighlight %}



#Locking

* Used for locking the entire table.

* One session cannot acquire locks for another session or release locks held by another session.


* UNLOCK TABLES explicitly releases any table locks held by the current session. 

* LOCK TABLES implicitly releases any table locks held by the current session before acquiring new locks.


* A table lock protects only against inappropriate reads or writes by other sessions. 




__READ__ [LOCAL] lock:   #[LOCAL] not valid for inno db

* The session that holds the lock can read the table (but not write it).

* Multiple sessions can acquire a READ lock for the table at the same time.

* Other sessions can read the table without explicitly acquiring a READ lock.



[LOW_PRIORITY] __WRITE__ lock:
 
* The session that holds the lock can read and write the table.

* Only the session that holds the lock can access the table. No other session can access it until the lock is released.

* Lock requests for the table by other sessions block while the WRITE lock is held.




A session that requires locks must acquire all the locks that it needs in a single LOCK TABLES statement. While the locks thus obtained are held, the session can access only the locked tables. 

WRITE locks normally have higher priority than READ locks to ensure that updates are processed as soon as possible. This means that if one session obtains a READ lock and then another session requests a WRITE lock, subsequent READ lock requests wait until the session that requested the WRITE lock has obtained the lock and released it. A request for a LOW_PRIORITY WRITE lock, by contrast, permits subsequent READ lock requests by other sessions to be satisfied first if they occur while the LOW_PRIORITY WRITE request is waiting.


{% highlight ruby %}
#session 1

mysql1> LOCK TABLES orders READ;
mysql1> select * from order_products;

Table 'order_products' was not locked with LOCK TABLES

mysql1> update orders set amount = 500 where id = 21548;

Table 'orders' was locked with a READ lock and can't be updated

#session 2
mysql2> LOCK TABLES orders WRITE;

#start waiting for the lock


#session 1
mysql1> UNLOCK TABLES; 
#wait over in connection 2

{% endhighlight %}


Update : Keep few points in mind, transactions are supported by storage engine InnoDB but not by MYIsam.And MyIsam supports table level locking while InnoDB doesn't.Althought MySql ensures table level locking on InnoDB but we should not use table level locks when using InnoDB storage engine.







