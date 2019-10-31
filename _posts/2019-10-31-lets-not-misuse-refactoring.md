---
layout: post
title: "Let's Not Misuse Refactoring"
date: 2019-10-31
categories: blog
tags: refactoring, good code
excerpt: >
  Refactoring has a specific meaning. When we misuse the word, we lose the
  ability to communicate an important concept. Let's revisit what refactoring is
  and what it is not.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/lets-not-misuse-refactoring">thoughtbot.com/blog/lets-not-misuse-refactoring</a>.
</div>


I find that many people confuse _refactoring_ with any change in code. Sometimes
they even use the word to mean huge changes that break the application &mdash;
"we need a month to refactor this monolith." I think that in those cases, we are
misusing the word refactoring, robbing it of its proper meaning, and ultimately,
robbing ourselves of the power to communicate an important concept.

Let's revisit what refactoring is and what it is not.

## The definition

Let's look at the definition of _refactoring_ from the [Refactoring] book<sup
id="a1"><small>[1](#f1)</small></sup>. Curiously, the word _refactoring_ has two
usages, though of course they are intimately related. One is a when we use the
word as a noun and the other when we use the word as a verb.

[Refactoring]: https://martinfowler.com/books/refactoring.html

When used as a noun:

> Refactoring (noun): a change made to the internal structure of software to
> make it easier to understand and cheaper to modify without changing its
> observable behavior.

This includes refactorings (noun) such as [Extract Method], [Rename Method], and
[Extract Class].

[Extract Method]: https://refactoring.com/catalog/extractFunction.html
[Rename Method]: https://refactoring.com/catalog/changeFunctionDeclaration.html
[Extract Class]: https://refactoring.com/catalog/extractClass.html

The second usage of refactoring is the verb form:

> Refactor (verb): to restructure software by applying a series of refactorings
> without changing its observable behavior.

So while we refactor (verb), we apply several refactorings (noun).

It is perhaps confusing if you haven't seen those definitions before, but one
point is clear across both definitions &mdash; refactoring is about changing the
structure of the code ***without*** changing its observable behavior.

So changing code is part of refactoring. But not all code changes are
refactorings. Code changes that alter the behavior of the code are not
refactorings, because refactoring is fundamentally about _not_ changing the
behavior of code.

## What if we break a public interface?

With such a definition, one can wonder how a refactoring (noun) such as Rename
Method even makes sense. If we change the name of a method, are we not changing
the behavior of the class? All code that calls the method will be
broken! Thankfully, the Refactoring book answers the question for us.

In the chapter _Problems with Refactoring_, the Refactoring book addresses the
question of a changing interface:

> There is no problem changing a method name if you have access to all the code
> that calls that method. Even if the method is public, as long as you can reach
> and change all the callers, you can rename the method. There is a problem only
> if the interface is being used by code that you cannot find and change. When
> this happens, I say that the interface becomes a published interface (a step
> beyond a public interface). Once you publish an interface, you can no longer
> safely change it and just edit the callers. You need a somewhat more
> complicated process.

I really like the distinction between a _public interface_ which we fully
control &mdash; and thus can refactor by changing all the callers of that
interface &mdash; and a _published interface_ which we do not fully control.
There, we cannot safely do a Rename Method refactoring because we would change
the observable behavior of the class.

## What exactly is the observable behavior of code?

If we cannot change the observable behavior of code, it is important that we
understand who is doing the observing. I find tests extremely helpful for this.
Since tests are ensuring the behavior of code does not change, we can use them
as the standard that we must keep while refactoring.

When we are refactoring the implementation of a single method in a class, then
not changing the behavior of the method means that the tests for that class
continue passing throughout our refactoring.

![Change internals of method. Unit tests still pass](https://images.thoughtbot.com/blog-vellum-image-uploads/Oo7qCPseRCCAaaO2H8rd_Refactoring-ChangeInternals.png)

If we have a larger change, however, where we rename a method and there are
repercussions throughout the code base, we need to find several tests (or a good
integration test) that cover all the code that would be affected. Sometimes that
may be the tests for the code that calls the method being renamed. If those
continue passing, then we have not broken the expected behavior.

![Rename method. Unit tests for calling classes still pass](https://images.thoughtbot.com/blog-vellum-image-uploads/znLqfBy5QKkTwKNnzyR4_Refactoring-RenameMethodTestCallingCode.png)

Other times, we may consider a feature test the standard of behavior we want
unchanged &mdash; making sure our users do not see a change in how a feature
works.

![Rename method. Feature test still passes](https://images.thoughtbot.com/blog-vellum-image-uploads/7hV74iTTd2rSlERQVv92_Refactoring-RenameMethodTestFeature.png)

So the observable behavior of code depends on who is doing the observing. And
that is why, once again, when we change a public interface &mdash; as long as
we control all the calling code &mdash; we can safely refactor. If we have a
published interface, however, we do not have control over the code that observes
the behavior of our code. And there we cannot safely refactor.

## Let's not rob ourselves of the word Refactoring

So not all changes of code are refactoring. Indeed, we cannot always refactor.
When we are talking about code changes that alter the behavior of our code,
let's not call it refactoring. Doing so robs the word of its meaning, and it
leaves us without a word to express a change in code that does not break its
behavior.

---

<small><b id="f1">1</b> All excerpts are taken from the first edition of the
book. [↩](#a1)</small>
