---
layout: post
title:  "Scope in Rails"
date:   2015-01-26
categories: blog
tags: [rubyonrails]
author: shobhit_garg
share: true
comments: true
excerpt:
---

Scope basically adds a __class method__ for __retrieving and querying objects__. A scope basically narrows the database query.You can use multiple scope in single staement.


{% highlight ruby %}
class Article < ActiveRecord::Base
  scope :published, -> { where(published: true) }
  scope :featured, -> { where(featured: true) }
end
{% endhighlight %}

We are able to call the methods like this:

Article.featured : This will return all the articles which are featured.(featured = true in databse)

Article.published.featured : This will return article whic are both published and featured.



All scope methods will return an ActiveRecord::Relation object which will allow for further methods or other scopes to be called on it.

{% highlight ruby %}
 class Article < ActiveRecord::Base
  scope :published,               -> { where(published: true) }
  scope :published_and_commented, -> { published.where("comments_count > 0") } #here published is an scope
end
{% endhighlight %}


__Passing a parameter in scope:__

{% highlight ruby %}
class Article < ActiveRecord::Base
  scope :created_before, ->(time) { where("created_at < ?", time) }
end
{% endhighlight %}

__Default scope:__


If you want to apply a particular condition in every query.Say you always want to fetch the active users from database then you can use:

{% highlight ruby %}
class User < ActiveRecord::Base
  default_scope { where status: 'active' }
end

User.all #=> User.where(:status => 'active') 
{% endhighlight %}


__Special feature::Conditional scope__ :

In case of false condition it would just fire the query without the scope:


{% highlight ruby %}
scope :by_status, -> status { where(status: status) if status.present? }

Post.by_status(nil).recent
# SELECT "posts".* FROM "posts" ORDER BY posts.updated_at DESC

Post.by_status('').recent
# SELECT "posts".* FROM "posts" ORDER BY posts.updated_at DESC

{% endhighlight %}







