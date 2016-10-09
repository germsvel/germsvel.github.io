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
describe BlogPost, '#publish' do
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

## Testing Private Methods

In the next posts on this series, I will look at how tests help with implementation and refactoring.
