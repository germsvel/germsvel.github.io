---
layout: post
title:  "Using merge in Rails to use an associated model's scope"
date:   2015-07-24 17:14:14
categories: blog
tags: Ruby, Rails, ActiveRecord
---

Everytime I want to create a Rails scope (read class method) that uses an associated model's scopes I have to search for this particular method.
Since I always forget what it is, I am finally writing a little post on using it.

Suppose you have two models, a `Blog` and a `BlogCategory`. Moreover, suppose we have blog posts for consumers and some for investors.
Suppose we only want to get the blogs that belong to consumers.

We could do something like this,

{% highlight ruby %}

Blog.joins(:blog_categories).where(category: 'consumer')

{% endhighlight %}

But it is nicer to put that in a scope (or a class method) so we define a nice interface by which we interact with our model.

So usually you might do something like `Blog.for_consumers`. We can achieve that by doing the following:

{% highlight ruby linenos=table %}
class Blog < ActiveRecord::Base
  has_many :blog_categories

  scope :for_consumers, -> { joins(:blog_categories).where(category: 'consumer') }
end
{% endhighlight %}

But what if we already have a scope in `BlogCategory` that does that? We'd like to use that.

{% highlight ruby linenos=table %}
class BlogCategory < ActiveRecord::Base

  scope :consumer, -> { where(category: 'consumer') }
end
{% endhighlight %}

Here comes `merge` to the rescue. We can just change our `Blog` model to use the scope.

{% highlight ruby linenos=table %}
class Blog < ActiveRecord::Base
  has_many :blog_categories

  scope :for_consumers, -> { joins(:blog_categories).merge(BlogCategory.consumer) }
end
{% endhighlight %}

Caveat: if the `Blog` and `BlogCategory` models both have the same attribute, that will throw an error because Rails will not know which model to retrieve the attribute for.
So if you have a `category` field in both `Blog` and `BlogCategory`, then make sure to define that in the `BlogCategory.consumer` explicitly.

{% highlight ruby linenos=table %}
class BlogCategory < ActiveRecord::Base

  scope :consumer, -> { where(blog_category: {category: 'consumer')} }
end
{% endhighlight %}

To find more info check out [apidock](http://apidock.com/rails/v4.0.2/ActiveRecord/SpawnMethods/merge).
