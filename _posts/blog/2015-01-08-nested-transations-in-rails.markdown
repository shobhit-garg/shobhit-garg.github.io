---
layout: post
title:  "Nested Transactions in Rails"
date:   2015-01-07
categories: blog
tags: [rubyonrails,transaction,database]
author: shobhit_garg
comments: true
share: true
excerpt: ""
---

In case of a single transation we can apply transation on one database.What if we need _transactions on multiple databases_? Rails supports nested transactions for this but before using it ,read about it carefully.

Suppose we have two databases A and B.You want to apply transactions on both the databases so that either two of these databases get updated together or nothing gets updated.

Case 1:

{% highlight ruby %}
A.transaction do
	B.transaction do
		#some operations on A and B both
		raise ActiveRecord::Rollback if (some condition)
	end
	#other operations
end
{% endhighlight %}

If you rollback the transaction this way,only the operations of database B are rolled back in this case becuase you are rollbacking the transactions of inner block.Other options are executed too.


Case 2:

{% highlight ruby %}
A.transaction do
	B.transaction do
		#some operations on A and B both
	end
	raise ActiveRecord::Rollback if (some condition)
end
{% endhighlight %}
If you rollback the transaction this way,only the operations of database A are rolled back because transaction block for B has alreday ended.

Case 3:

{% highlight ruby %}
A.transaction do
#some operaitons on A
raise ActiveRecord::Rollback if (some condition)
	B.transaction do
		#some operations on A and B both
	end
end
{% endhighlight %}

In this case operations related to A are rolled back and system does not execute B's transaction block.


Case 4(A valid way):

{% highlight ruby %}
#A valid way
begin
A.transaction do
	B.transaction do
		#some operations on A and B both
		raise Exception if (some condition) #you can create your own exception for raising
	end
end
rescue Exception=>e
Rails.logger.info {"Both the transactions rollbacked."}
end
{% endhighlight %}

In this case _both the transactions are rollbacked_.

If you want to rollback both the transactions using _ActiveRecord::Rollback_ you have to use _raise ActiveRecord::Rollback_ in both inner and outer block.

Check out [this blog post][blg-post],you will find it helpful.

[blg-post]:		http://markdaggett.com/blog/2011/12/01/transactions-in-rails/


