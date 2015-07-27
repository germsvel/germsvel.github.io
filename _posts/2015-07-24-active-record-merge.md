---
layout: post
title:  "Merging an associated model's conditions into your scope"
date:   2015-07-24 17:14:14
categories: blog
tags: Ruby, Rails, ActiveRecord
---

Everytime I want to create a Rails scope that uses an associated model's scopes I have to search for this particular method.
Since I always forget what it is, I am finally writing a short post on using it.

Suppose you have two models, a `Blog` and a `BlogCategory`. Some blogs are for consumers and some for investors.
If we want to get the blogs that belong to consumers, we could do something like this

{% highlight ruby %}

Blog.joins(:blog_categories).where(blog_categories: {category: 'consumer'})

{% endhighlight %}

But it is better to put that in a scope so that we can reuse it and so that we create a nicer interface to interact with our model:

{% highlight ruby linenos=table %}

class Blog < ActiveRecord::Base
  has_many :blog_categories

  scope :for_consumers, -> { joins(:blog_categories).where(blog_categories: {category: 'consumer'}) }
end

{% endhighlight %}

But what if we already have a scope in `BlogCategory` that we'd like to use?

{% highlight ruby linenos=table %}

class BlogCategory < ActiveRecord::Base
  belongs_to :blog

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

Caveat: if the `Blog` and `BlogCategory` models both have the same attribute (say `category`), and you are using pure string conditions in your where clause like this

{% highlight ruby linenos=table %}
class BlogCategory < ActiveRecord::Base

  scope :not_consumer, -> { where("category != 'consumer'") }
end

{% endhighlight %}

and our `Blog` is using that scope,

{% highlight ruby linenos=table %}
class Blog < ActiveRecord::Base
  has_many :blog_categories

  scope :for_non_consumers, -> { joins(:blog_categories).merge(BlogCategory.not_consumer) }
end
{% endhighlight %}

then trying to use `Blog.for_non_consumers` will give us an `ActiveRecord ambiguous column name` error. That happens because Rails does not know which table to use for the `category` column.

In order to fix that, make sure to explicitly define which model is supposed to use that column:

{% highlight ruby linenos=table %}
class BlogCategory < ActiveRecord::Base

  scope :not_consumer, -> { where("blog_categories.category != 'consumer'") }
end
{% endhighlight %}

Then we can safely use `Blog.for_non_consumers`.

To find more info check out [apidock](http://apidock.com/rails/v4.0.2/ActiveRecord/SpawnMethods/merge).
