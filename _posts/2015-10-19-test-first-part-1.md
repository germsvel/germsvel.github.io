---
layout: post
title:  "Test First - Part 1"
date:   2015-10-19
categories: blog
tags: Ruby, RSpec, Testing
published: false
---

After working with some co-workers who do not practice test-driven development, I was struck by how much my worflow depends on tests.
It was only then that I realized just how much tests are an integral part of my work. And I don't mean writing tests as an afterthought
because the team requires it to do so. But rather, relying on your tests for so many things. Anyway, since I came to that realization
recently, I have decided to put in text some of my workflow. Perhaps it is more of mental flow. In any case, when I say test-first,
I do not only mean TDD or BDD, though I certainly don't mean less. But rather, when I say test-first I mean a whole workflow that considers
tests as first class citizens and as drivers of understanding, exploration, design, implementation, and refactoring.

I imagine this will take a few blog posts, so I start with part-1, hoping that more parts will follow. I will progress as I might progress
when writing a feature. In the process, I will give some insight into how I tend to think, and how involved tests are in that thinking
process. It is my hope that some may be helped by seeing how my workflow is influenced by tests, and that it would encourage them to test
more, or at all, if they haven't yet discovered the amazing power of keeping your tests first.

## Understanding

Whenever I start working on a feature, I seek to understand the flow of code. Most of my time is spent working on Rails apps, so my flow
will naturally go from a view or a controller down the stack. During this process, I often encounter pieces of code I have not seen
before. When I do so, I love running the classes tests with documentation format. In `rspec`, you can get that by running,

```
rspec spec/class_name_spec.rb --fd
```

I find this extremely helpful. A caveat is that this only works well if the tests are well written. Sometimes I find that the description
of the test does not correspond to what the test is actually asserting. It is very helpful when the `describe`, `context`, `it` blocks are
well presented. I tend to prefer having a `describe` for every public method a class exposes. That way, when you look at tests as documentation
you can see very clearly what is the public interface of the class and presumably what are its return values.

This is a good example,

```
include example of nice well documented tests here. Perhaps include the rspec command with --fd flag.
```

This is a bad example,

```
include example of poorly written test descriptions here.
```

You can see that with the good test descriptions, you can immediately get a fair understading of what the class is suppposed to do. I love finding
these kinds of tests as I try to understand the flow of code. Unfortunately, you do not always find good test descriptions, if you find them at all.


## Exploring

When I find a class that has incomplete tests or no tests at all, I typically like to add good tests to it. I say _typically_ because I understand that
it may not always be the best use of time. I usually have to make a decision based on how relevant the class is for the work I need to do. If I can
understand well enough what the code is doing, and it is not the most important class, I may opt to not write tests at the moment. I may come back if I
have time at the end, but at the moment there's too many unknowns to know if I have time to write tests.

Coincindentally, this understanding of my own trade-offs has shed some light (I think) on why people who don't like to write tests-first can think of
tests as a slowing, painful thing to do. My guess (and it is a guess) is that they write all their code first, and having completed the functionality,
they go on to write some tests. They then have to write tests for classes they feel are not worth the tradeoff that I explained. And surely they must
feel like this is taking twice as long. It is for them! The tests are not built into the way they process how to implement a feature. So tests are just
this added nuisance that we have to maintain when they break. They don't really offer any other benefit, aside perhaps from seeing the entire suite
pass before deployment.

Back to exploring. If I find that I cannot easily understand what the class is doing, or if perhaps I partially understand but I feel the class will
play an intricate role in the feature I am writing, then I will use tests as a learning tool, tests in exploratory mode.

I think people do this anyway. When they don't understand the behavior of a class, they may put a `binding.pry` and open up their browser to see if
they can hit the breakpoint. Or perhaps they stare at the methods for a long time, untangling the difficult to read method signatures. A class that
is difficult to understand is difficult to understand to those who write tests-first and those who do not alike. And as such, brain power will be
exerted and time will the used in order to understand it. But I think here tests can be a huge help.

Why put a breakpoint and open up the browser when you can just write a test and see what the class does? You don't even have to know what the class
is supposed to do in order to write a good assertion. In fact, I often write assertions I know will not evaluate to true in order to see what kind
of error I get.

For example, I may write something like this,

{% highlight ruby linenos=table %}
describe BlogPost, '#complex_method' do
  it 'works' do
    post = BlogPost.new

    response = post.complex_method

    expect(response).to eq 'hello'
  end
end
{% endhighlight %}

That test will fail, unless that complex method actually returns `'hello'`, which is highly unlikely. In its failure, I will gain understanding of what
the class actually does. Say it returns an error because it needs a `title`.

```
show an actual error here to demonstrate that error messages are very helpful.
```

This points us to something very important lesson about tests: listen to the error messages. By listening to the error message, I have now learned that a
blog post needs a title. And sure, we could have learned that by looking at the class itself, which in fact I would likely have open in a different pane
right next to my tests. But if that error came from the very confusing method, or if it was an error that you did not expect at all because some class
in the inheritance chain threw an error, you'd learn that much faster than via any other method.

So then we migth update our test to include the required title and run it again,

{% highlight ruby linenos=table %}
describe BlogPost, '#complex_method' do
  it 'works' do
    post = BlogPost.new(title: 'test first part-1')

    response = post.complex_method

    expect(response).to eq 'hello'
  end
end
{% endhighlight %}

And now we get a new message,

```
Put error message of expected 'hello' got nil
```

Now we have learned something else. We now know that if you just provide a `title` invoking that method does not break anything, but it also doesn't
seem to return anything useful. This may not seem useful to people, but I find it a very useful piece of information. It tells me that the default
behavior of the method is to return `nil`. And some may think this an unrealistic example, but I assure you I have found this very thing time and
time again in code. Naturally there are a lot of questions I may not be able to answer regarding this return value without asking the intial
writers of the method or without more in-depth research, but I think it is important to document that the default return value is `nil`. To drive
this point home a little bit more, suppose that `#complex_method` was just `#body`. Then (a) you may be a little puzzled to see `nil` instead of
an empty string, and (b) you may be glad to learn that's the default behavior of the body. So, I would update the description of the first test
to reflect my new understanding of the class and then I would write a second test,


{% highlight ruby linenos=table %}
describe BlogPost, '#body' do
  it 'returns nil by default' do
    post = BlogPost.new(title: 'test first part-1')

    default_body = post.body

    expect(default_body).to be_nil
  end

  it 'returns the body if provided?' do
    post = BlogPost.new(title: 'test first part-1', body: 'this is the body')

    post_body = post.body

    expect(post_body).to eq 'this is the body'
  end
end
{% endhighlight %}


If the second test passes, then I have learned some valuables things about the class and have docuented the behavior of its public interface.
If the test does not pass, and instead I get an error like this,

```
write an error that makes sense. Expected 'this is the body' but received '<p> this is the body</p>' instead.
```

Then I will repeat the process described above. I would update the test description to reflect what that tests actually does and keep adding
tests.

As a last note of exploration, I think this method of documenting as I go along is extremely valuable not only for me but for those who come
after me. If this is a method that is difficult to understand, then I know it takes time and brain power for a developer to understand in order
to be able to implement whatever feature I'm working on. But once I understand this method, I should document my undestanding somewhere so that
the next developer who comes along and needs to understand it can be guided by my documentation. I think having those tests explain the public
interface and its return values is an excellent way of doing that. In fact, having created those tests will allow me to refactor the complex
method and make it easier to read, if I have time in my hands. But even if I don't, the person after me may come along and have the time to
refactor the method since they don't have to write the tests and since they can more readily understand the code.

It is hard to keep that in mind. I think we tend to think of programming as somewhat of a solo sport. And yet, I think having a mindfulness for
other developers who will come along the code you touch is a very important aspect of being a good programmer. And in that sense, we are a playing
a team sport, trying to make code more and more clear, readable, and concise. I know I appreciate when I find a great piece of code with amazing
tests and easy to understand. It is often in those cases that I do not even think of the developers that came before me and worked on this code.
Before I have time to think about them, I am on to the next class. And I think that's a good thing.

Stay tuned for the next parts of this test first series. We just scratched the surface with documentation and exploration. I still want to look
to design, implementation, and refactoring.







