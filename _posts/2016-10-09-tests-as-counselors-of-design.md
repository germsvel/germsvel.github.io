---
layout: post
title:  "Tests as Counselors of Design"
date:   2016-10-09
categories: blog
tags: Ruby, RSpec, Testing
excerpt: <p>One of the benefits of tests is that they are the first piece of code that will interact with our application. As such, they can offer valuable insights into the application's complexity.</p>
---


It was only recently that I realized just how much tests are an integral part of
how I code. I use them constantly, allowing them to influence every phase of my
workflow, so I thought I'd share some ways in which they help me.

This blog post is part of a series called **Test First Workflow** that came out
of a talk I gave last year. I identified four major ways in which tests help me:
tests are tools for understanding, counselors of design, drivers of
implementation, and pillars of refactoring. I will write a blog post for each
of them. This is part two.

# Counselors of Design

One of the benefits of tests is that they are the first piece of code that will
interact with our application. As such, they can offer valuable insights into
the application's complexity. But this only comes when we recognize tests as
an important aspect of our workflow. If we merely think of tests as an
afterthought, we are likely to miss this important benefit of testing.

## Hard to Test

One way in which our tests counsel us about code complexity in our application is by
being complex themselves. When a test is difficult to write, it usually means there
are either too many dependencies or that the code we are testing has complex branch
logic that is hard to follow.

## Too Much Setup

When a test needs lots of setup, it may be a sign that the code we are
trying to test requires lots of dependencies in order to function properly. In
other words, we have code that is very coupled.

In object-oriented programming, we do not want a class to be coupled to many
other classes. In general, the looser the coupling the easier it is to reuse the
objects and to extend our application, and that's a good thing.

So if you see something like this in your tests then they may be telling
you that some of your classes are too coupled.

{% highlight ruby %}
RSpec.describe BlogPost, '#publish' do

  it 'publishes a blog post' do
    pages = Pages.create(20)
    Blog.create(pages)
    author = Author.create
    editor = CopyEditor.create
    editor.authors << author
    Platform.create_with_author(author)

    post = BlogPost.new(title: 'test first part-2')

    post.publish

    expect(post).to be_published
  end

end
{% endhighlight %}

A notable exception to the too-much-setup code smell is a feature spec. Usually,
feature specs require more setup since they are doing integration testing. Even
then, however, your tests may be telling you that there is too much coupling
between classes. It is just harder to know whether the large setup is due to
integration testing or the product of strong coupling.

## Too Many "Contexts"

In `RSpec`, we define a group of related tests by keeping them all within
a `describe` or a `context` block. For example, one might see the following spec,

{% highlight ruby %}
RSpec.describe BlogPost do
  context 'with author' do
    it 'is publishable' do
      # some code here
    end
  end

  context 'without author' do
    it 'is unpublishable' do
      # some code here
    end
  end
end
{% endhighlight %}

Naturally, not every use of `context` blocks is bad. But they are often used as
a consequence of branching logic in the method being tested. For example, say we
are testing the following method,

{% highlight ruby %}
class Blog
  def cancel_post(post)

    case post.status
    when :published
      # do a
    when :draft
      # do b
    else
      # do c
    end

  end
end
{% endhighlight %}

Our spec file looks like this,

{% highlight ruby %}
RSpec.describe Blog, '#cancel_post' do

  context 'when post status is published' do
    it 'cannot be canceled' do
      # test here
    end
  end

  context 'when post status is draft' do
    it 'is leaves it as draft' do
      # test here
    end
  end

  context 'when post status is something else' do
    it 'deletes record' do
      # test here
    end
  end
end
{% endhighlight %}

Those familiar with code smells will have already noticed that case statements
are a code smell in and of themselves. For those who are less familiar with code
smells, case statements often reveal that we have missed an abstraction. A good
[remedy](https://sourcemaking.com/refactoring/smells/switch-statements) is to
use polymorphism. So we see that tests can tells us, like a good counselor
might, that the method we are testing could use some design help.

A less obvious example may be a method with an `if/else` statement in it.
Many people would just look at a method with a conditional statement and think
nothing wrong of it. But what holds true for the `case` statement holds true
for an `if/else` statement. We are often overlooking an abstraction.

## Testing Several Responsibilities

A second counsel the a test with many `context` blocks may give us comes
when the `context` blocks are not at the method level but at the class level.
For example, in the following test we have `context` blocks helping us separate
two different things the blog is doing. It may even help us keep our tests DRY
by allowing us to set up `before` blocks, lulling us into a false sense of code
cleanliness.

{% highlight ruby %}
RSpec.describe Blog do
  context 'with articles' do
    before do
      # setup for articles
    end

    it 'only publishes online articles' do
      # test here
    end
  end

  context 'when sending articles via email ' do
    before do
      # setup for recipients
    end

    it 'only sends posts that are published' do
      # test here
    end
  end
end
{% endhighlight %}

And say our class looks something like this,

{% highlight ruby %}
class Blog
  def initialize
    @articles = []
    @recipients = []
  end

  def publish_articles
    @articles.each do |article|
      if article.online?
        article.publish
      end
    end
  end

  def send_articles
    @recipients.each do |recipient|
      send_articles(recipient)
    end
  end

  private

  def send_articles(recipient)
    # ...
  end
end
{% endhighlight %}


This may seem normal to some, after all how many times have seen things like
these in our code bases?

{% highlight ruby %}

blog = Blog.find(id: blog_id)

blog.publish_articles

blog.send_articles

{% endhighlight %}


But those familiar with the [Single Responsibility
Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) know
that it is good to ask, "What is the object doing? What is it's reponsibility?"
When the answer to that question comes with an "and", we can conclude that the
class has more than one responsibility and hence more than one reason to change.
For example, the answer to our code above may be "the blog is publishing
articles _and_ it is sending those articles via email". Once again our tests act
like good counselors and tell us that our class has too many responsibilities.

## Testing Private Methods

Another way in which tests counsel us with the design of our code is when we
test or want to test a private method.

For example, we may see something like this,

{% highlight ruby %}
RSpec.describe Blog, '#slug' do

  it 'dynamically creates a slug from the author’s name' do
    blog = Blog.build

    slug = blog.send(:create_slug)

    expect(slug).to eq 'the-greatest-author-name'
  end

end
{% endhighlight %}

Using ruby's `send` method allows you to invoke methods regardless of whether
they are private or not.

But testing private methods is usually done because they have important or
complex logic that we want tested in our application. And anything that is that
important would better serve our application as a public method in another class
or as part of the public interface of a new object. The fact that we want to
test it should help us recognize that this code has an important function
in our application and is not something to be relegated to a private method.

For example, in our example above, the private method probably
belongs in the `Author` class, and our test becomes a test of a public method,

{% highlight ruby %}
RSpec.describe Author, '#create_slug' do

  it 'dynamically creates a slug from the name' do
    author = Author.new

    slug = author.create_slug

    expect(slug).to eq 'the-greatest-author-name'
  end

end
{% endhighlight %}

There are other ways in which our tests counsel us towards better design. I have
only here listed some. But as a general rule, if you find that the test is hard to
write, look first towards what might be wrong with the design of the code, not
towards what might be wrong with the test itself. People who don't like testing
often want to blame the test. But don't shoot the messenger; fix the real
problem.

Up next in this series will look at how tests help with implementation
and refactoring. Stay tuned.
