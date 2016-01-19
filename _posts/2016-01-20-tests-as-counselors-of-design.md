---
layout: post
title:  "Tests as Counselors of Design"
date:   2016-01-18
categories: blog
tags: Ruby, RSpec, Testing
published: false
---

>> _This blog post is part of a series called **Test First Workflow** that came out of a talk I gave last year.
I will write a separate blog post for each of the four ways I use tests in my workflow. This is part two._

It was only recently that I realized just how much tests are an integral part of how I code.
I use them constantly, allowing them to influence every stage of my workflow, so I thought I'd share some ways in which they help me.

I'll break the ways tests are part of my workflow into four camps: tests are tools for understanding, counselors of design, drivers of implementation, and pillars of refactoring.

# Counselors of Design

One of the great benefits of tests is that they are the first piece of code that interacts with our application.
They are the first "users" of our code. So we can learn about the design of our code from what they tells us.

Just as in our regular code there are code smells that can lead us to reconsider the design of the code, tests that are difficult to write have smells that point to
flaws in design in our code. Let's look at a few:

1. Mystery guests needed for test to pass

This smell often comes when we are trying to write a test for an _already existing_ method, but the test keeps failing and we keep needing to add setup to our test example.
For example, suppose we are testing a blog post's `publish` method. We might expect to be able to write something like this and have the test pass,

{% highlight ruby %}
RSpec.describe BlogPost, '#publish' do
  it 'publishes a post' do
    post = BlogPost.create(title: 'Design', body: 'Tests are awesome for design')

    post.publish

    expect(post).to be_published
  end
end
{% endhighlight %}

Suppose the test fails `expected post.published? to return true but returned false instead`.

Looking at the implementation of the code, we may see that the method is doing a type check,

{% highlight ruby %}
  def publish
    if self.type == 'public'
      ##
    end
  end
{% endhighlight %}

So we go back to our test and add the type,

{% highlight ruby %}
RSpec.describe BlogPost, '#publish' do
  it 'publishes a post' do
    post = BlogPost.create(title: 'Design', body: 'Tests are awesome for design', type: 'public')

    post.publish

    expect(post).to be_published
  end
end
{% endhighlight %}

But running the test again, the test fails. Looking further into our implementation we see,

{% highlight ruby %}
  def publish
    if self.type == 'public'
      return unless has_content?
    end
    ##
  end
{% endhighlight %}

So maybe, we reason, we want a blog post that has some content. We add that condition to our test,

{% highlight ruby %}
RSpec.describe BlogPost, '#publish' do
  it 'publishes a post' do
    post = BlogPost.create(title: 'Design', body: 'Tests are awesome for design', type: 'public')
    post.has_content!

    post.publish

    expect(post).to be_published
  end
end
{% endhighlight %}

And the test fails, and so it goes on an on. What the test is telling us is that the method we are testing has a complex structure, which is a code smell.
Each method should be doing one thing. Instead this method is doing a lot of things.
