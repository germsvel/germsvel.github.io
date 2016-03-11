---
layout: post
title:  "Tests as Counselors of Design"
categories: blog
tags: Ruby, RSpec, Testing
---

>> _This blog post is part of a series called **Test First Workflow** that came out of a talk I gave last year.
I will write a separate blog post for each of the four ways I use tests in my workflow. This is part two._

It was only recently that I realized just how much tests are an integral part of how I code.
I use them constantly, allowing them to influence every stage of my workflow, so I thought I’d share some ways in which they help me.

I’ll break the ways tests are part of my workflow into four camps: tests are tools for understanding, counselors of design, drivers of implementation, and pillars of refactoring.

# Counselors of Design

One of the great benefits of tests is that they are the first piece of code that interacts with our application.
They are the first "users" of our code. So we can learn a lot about the design of our code from what they tells us.

Just as in our regular code there are code smells that can lead us to reconsider the design of the code, tests that are difficult to write have smells that point to
flaws in design in our code. Let’s look at a few:

1. When a test requires too much setup

When tests have or require a lot of setup, it is often indicative of tight coupling between the class being tested and other classes.
For example, looking at our blog post example, if we needed to setup three other objects to `publish` the blog post, a blog, and author, and an editor,
then we know that our `BlogPost` class depends on those other classes for its functioning. Some coupling is perhaps unaviodable in the case of a `BlogPost`, but most
of the times we want our code to be lightly coupled so that it is easily extensible, and so that changes in one class do not necessitate changes in other classes.

{% highlight ruby %}
RSpec.describe BlogPost, '#publish' do
  it 'publishes a post' do
    blog = Blog.create
    author = Author.create
    editor = Editor.create
    post = BlogPost.create(blog: blog, author: author, editor: editor)

    post.publish

    expect(post).to be_published
  end
end
{% endhighlight %}

Of course, as with any code smell, it does not mean this is necessarily a problem or that it is something that needs to be fixed.
For example, we can expect feature specs to have more setup since they have to run through a larger section of our code.

2. Too many `context`s
3. Testing private methods

