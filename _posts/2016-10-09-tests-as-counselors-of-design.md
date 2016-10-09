---
layout: post
title:  "Tests as Counselors of Design"
date:   2016-10-09
categories: blog
tags: Ruby, RSpec, Testing
excerpt: <p>One of the benefits of tests is that they are the first piece of code that will interact with our application code. As such, they can offer valuable insights into the complexity of our code.</p>
---


>> _This blog post is part of a series called **Test First Workflow** that came out of a talk I gave last year.
I will write a separate blog post for each of the four ways I use tests in my workflow. This is part two._


It was only recently that I realized just how much tests are an integral part of how I code.
I use them constantly, allowing them to influence every stage of my workflow, so I thought I'd share some ways in which they help me.

I'll break the ways tests are part of my workflow into four camps: tests are tools for understanding, counselors of design, drivers of implementation, and pillars of refactoring.

# Counselors of Design

One of the benefits of tests is that they are the first piece of code that will
interact with our application code. As such, they can offer valuable insights
into the complexity of our code.

If we think of tests merely as an afterthought,
something that we _have_ to do only because someone in our team is pushing for
it, we are likely to miss this important benefit of testing. But if we see test
as first-class citizens, then we can look at them not just as something that is
ensuring that our code works but also as the first "user" of our application.

In other words, tests can behave something like [code
smells](https://en.wikipedia.org/wiki/Code_smell).

## Hard to Test

One way in which our tests point us to code complexity in our application is by
being complex themselves. When a test is hard to write, it usually means there
are either too many dependencies or that the code we are testing has complex branch
logic that we can't quite follow correctly.

### Too Much Setup

A code smell that is easy to spot is that of too much setup. When a test
requires lots and lots of setup, the tests are telling us that the code we are
trying to test has lots and lots of dependencies that are required, which means
we have code that is very coupled.

In object-oriented programming, we do not want a class to be coupled to many
other classes. The looser the coupling, the easier it is to reuse the objects
and to extend our application, and that's a good thing.

So if you see something like this in your tests,

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

then your test may be telling you that there are a lot of dependencies needed to
create and publish a blog post. Perhaps things are too coupled.

A notable exception to the too-much-setup code smell is a feature spec. Usually,
feature specs may require more setup since they are testing integrations.
Nevertheless, even then your tests may be telling you that there is a lot of
coupling between classes. It may just be harder to tell whether or not it is
mere integration test setup or a product of strong coupling.

### Too Many "Contexts"

In `RSpec`, the way to define a group of related tests is by keeping them all within
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

Now `context` blocks are not a problem in and of themselves, and not every use of
them is bad. But they are often used as a consequence of branching logic in the
method being tested. For example, say we are testing the following method,

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

We may then have our spec file look like this,

{% highlight ruby %}
RSpec.describe Blog, '#cancel_post' do

  context 'when post status is published' do
    # test here
  end

  context 'when post status is draft' do
    # test here
  end

  context 'when post status is something else' do
    # test here
  end
end
{% endhighlight %}

Those already familiar with code smells might have already noticed that case
statements are code smells in and of themselves. But others may not have been
so trained, so the test tells us, like a good counselor might, that our method
probably has more than one responsibility.

A less clear example may be a method with an `if/else` statement in it.
Many people would just read that and not think twice.  But those familiar with
the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle)
know that branching logic, even if only an `if/else` statement, can mean that
the objet has more than one respnosibility. The object is doing the `if`
part _and_ it is doing the `else` part. The "and" hints at multiple
responsibilities and so do multiple `context` blocks in a test.

## Testing Private Methods

Another way in which tests counsel us in the design of our code is when we
either want to write a test for a private method or see a test that is already
testing a private method.

For example, you may see something like this,

{% highlight ruby %}
RSpec.describe Blog, '#slug' do
  it 'dynamically calculates a slug from the author’s name' do
    blog = Blog.build

    slug = blog.send(:create_slug)

    expect(slug).to eq 'the-greatest-author-name'
  end
end
{% endhighlight %}

Using ruby's `send` method allows you to invoke methods regardless of whether
they are private or not.

But testing private methods is usually done because they have important or
complex logic that we want to be tested in our system. And anything that is that
important should be extracted into public methods in other classes or into an
object of its own. Testing should only be done of public methods.

For example, in our example above, it may be that the private method really
belongs in the `Author` class, and our test would then test a public method,

{% highlight ruby %}
RSpec.describe Author, '#create_slug' do

  it 'dynamically calculates a slug from the name' do
    author = Author.new

    slug = author.create_slug

    expect(slug).to eq 'the-greatest-author-name'
  end
end
{% endhighlight %}


___

In the next posts on this series, I will look at how tests help with implementation and refactoring.
