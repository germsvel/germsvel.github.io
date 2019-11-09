---
layout: post
title: "Finding the Time to Refactor"
date: 2019-11-11
categories: blog
tags: refactoring, good code
excerpt: >
  Many people ask, "How do I find time to refactor?" I think the question itself
  betrays a misunderstanding of refactoring. Let me tell you when I refactor.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/finding-the-time-to-refactor">thoughtbot.com/blog/finding-the-time-to-refactor</a>.
</div>

Many people ask, "How do I find time to refactor?" I think the question itself
betrays a misunderstanding of refactoring. The solution, therefore, does not lie
in finding ways to convince a product manager to schedule "refactor time" but in
changing our way of thinking. Let me tell you when I refactor.

## Thinking differently

Some people think that programming is about making the code work. And at a basic
level, it is. But programming is about so much more.

Programming is about managing an ever-changing code base. It is about satisfying
concrete requirements today while being able to accept new requirements
tomorrow. It is about being able to clearly communicate your intent across space
and time to other developers who will read, modify, and extend your code. For
that reason, programming is not about just writing code that works. It is about
writing code that is easy to read, easy to reason about, and easy to modify.

### Redefining being "done"

Getting the code to work is required but not sufficient. In test-driven
development, we have the [red-green-refactor cycle]. Getting the code to work
means going through red and green &mdash; getting the tests to pass. But
refactoring is about that last step &mdash; about making sure the code we just
introduced is clear and maintainable.

![Diagram showing red, green, refactor cycle. It marks refactor as done](https://images.thoughtbot.com/blog-vellum-image-uploads/RcclOB9nRC6ejvnAWXk5_finding-time-refactor-tdd-cycle.png)

This is where the change in mindset comes in. As developers, we should think of
our work as being "done" not just when the code works but when the code works
_and_ it is clear and maintainable.

That is why refactoring should happen all the time. There is no need to
introduce a "refactor time" in the schedule. It is baked into the very
definition of doing work.

[red-green-refactor cycle]: https://thoughtbot.com/upcase/videos/red-green-refactor-by-example

## Clarifying what refactoring is

Now, some might be hesitant to refactor all the time because they think it means
making large breaking changes. But [that's not what refactoring is]. At
its core, refactoring is about changing the structure of code _without_ changing
its observable behavior. Take a look at the definition of refactoring from the
[Refactoring] book:

> Refactoring (noun): a change made to the internal structure of software to
> make it easier to understand and cheaper to modify without changing its
> observable behavior.

So while we work, we should constantly try to improve the code without changing
the behavior _so that_ the code is easier to understand or cheaper to modify.

[that's not what refactoring is]: http://www.germanvelasco.com/blog/2019/10/31/lets-not-misuse-refactoring.html

## When to refactor

> You don't decide to refactor, you refactor because you want to do something
> else, and refactoring helps you do that other thing.
>
> &ndash; Martin Fowler in Refactoring: Improving the Design of Existing Code

In practice, there are usually three instances when I refactor: as the last
step of red-green-refactor, when trying to modify code that is difficult to
understand, and when an existing design doesn't work.

### 1. The last step in red-green-refactor

The first instance is when I'm test-driving a feature. As mentioned before, the
red-green-refactor cycle has refactoring built in. So when I get the tests to
pass, I take a few minutes to clean up any code I have introduced: use better
names, extract private methods, extract classes, etc.

### 2. Code that is difficult to understand

When I am trying to add a feature or fix a bug, but I have to modify an
existing piece of code that is difficult to understand, I refactor it (once I
understand what it is doing). That way, I make sure I am introducing the
correct change into the code base, and others in the future (myself
included) will more easily understand it. We pay the tax of difficult-to-read
code once, not every time someone has to read that code.

### 3. Existing design doesn't work

Finally, I refactor when I'm trying to introduce something new, but it does
not quite fit the existing design. I first refactor the design so that what I am
trying to introduce is easy to add. This is what Martin Fowler calls
[Preparatory Refactoring]. In the [Refactoring] book, he says:

> When you find you have to add a feature to a program, and the program's code is
not structured in a convenient way to add the feature, first refactor the
program to make it easy to add the feature, then add the feature.

Or as it is commonly said, ["Make the change easy, then make the easy
change"](https://twitter.com/kentbeck/status/250733358307500032).

## What next?

If you're interested in practicing refactoring on a complicated piece of code, I
recommend trying out the [Gilded Rose Kata]. It's a very fun and illuminating
exercise.

And if I communicate nothing else, I highly recommend reading the [Refactoring]
book. It's full of great information on how, when, and why to do refactoring.

[Refactoring]: https://martinfowler.com/books/refactoring.html
[preparatory refactoring]: https://martinfowler.com/articles/preparatory-refactoring-example.html
[Gilded Rose Kata]: https://github.com/jimweirich/gilded_rose_kata
