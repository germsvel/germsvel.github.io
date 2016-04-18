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

In this post we’ll look at how they are counselors of the design.

# Counselors of Design

One of the great benefits of tests is that they are the first piece of code that interacts with our application.
They are the first "users" of our code. So we can learn a lot about the design of our code from what they tells us.

Just as in our regular code there are code smells that can lead us to reconsider the design of the code, tests that are difficult to write have smells that point to
flaws in design in our code. Let’s look at a few:

1. When a test requires too much setup

When tests have or require a lot of setup, it is often indicative of tight coupling between the class being tested and other classes.
Decoupled code makes our application more easily extensible, making it so that changes in one class do not necessitate changes in several other classes.

For example, looking at a test example below, in order to `publish` a blog post, we need to set up a blog, an author, and an editor.
This signals thatn in one way or another, our `BlogPost` class _depends_ on those other classes for its functioning.

{% highlight ruby %}
RSpec.describe BlogPost, '#publish' do
  it 'publishes a post' do
    Blog.create
    Author.create
    Editor.create
    post = BlogPost.create

    post.publish

    expect(post).to be_published
  end
end
{% endhighlight %}

It may be that if we looked at the `publish` method in the `BlogPost` class we would find the `Author`, `Editor`, and `Blog` classes referenced. Or it might be that
one of those classes is referenced in `publish` but the other two are referenced within that class. Whatever the case may be, this test is telling us that there is
a tight coupling between these classes. Whether the coupling is intentional or necessary, the test cannot tell us. But it serves as a wise counselor revealing a possible
design flaw.

2. Too many `context`s

When we see a test with several contexts, it is often an indication that the code it is testing has conditional logic.
For example, we may see a test like this,

{% highlight ruby %}
RSpec.describe InsuranceProvider, '#cost' do
  context 'Allstate' do
    ...
  end

  context 'Esurance' do
    ...
  end

  context 'Geico' do
    ...
  end

  ...

end
{% endhighlight %}

It would not be surprising to find a `case` statement in the implementation of `cost`. E.g.,

{% highlight ruby %}
class InsuranceProvider
  ...

  def cost(provider)
    case provider
    when 'Allstate'
      250
    when 'Esurance'
      300
    when 'Geico'
      200
    ...
    else
      0
    end
  end
end
{% endhighlight %}

Just as `case` statements are a code smell that we are missing an abstraction, so do too many `contexts` imply that we are
missing an abstraction. In both cases, we know could refactor the code to use polymorphism.

It is good to note that just as not all case statements need to be changed (sometimes a case statement is desired), a test having
lots of `context` blocks or `describe` blocks does not always point to a missed abstraction. Still, as wise counselors, our tests
inform us that something may be fishy.

3. Testing private methods

Private methods should be... well private.

