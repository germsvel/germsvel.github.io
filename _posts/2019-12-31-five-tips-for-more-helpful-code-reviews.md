---
layout: post
title: "5 Tips for More Helpful Code Reviews"
categories: blog
tags: code review, process
excerpt: >
  Having your code reviewed can be daunting. But it can also be very helpful. As
  reviewers, we can make the difference. Here are five tips to make your code
  reviews more helpful to the author.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/five-tips-for-more-helpful-code-reviews">thoughtbot.com/blog/five-tips-for-more-helpful-code-reviews</a>.

</div>

Submitting code for review can be daunting. After all, it is exposing our work
to critique, and that can be terrifying regardless of the industry. But code
review can also be a great tool for learning and improvement. As reviewers, we
can make all the difference. Here are five things I do when reviewing code to
try to make it as helpful as possible for the author.

## Rewrite comments for empathy and clarity

In code reviews, a comment is interpreted by _how_ the author reads it.
Therefore, as a reviewer, you should consider the potential impact of your
words, not just their original intent. Otherwise, you may be surprised to find
that a comment you meant to be helpful was merely hurtful.

That's why rewriting comments until they clearly convey your meaning with
empathy is very important. Sometimes we act as though writing comments is like
drawing with a permanent marker: what goes on the first time is the final
product. But the opposite is true &mdash; most good comments are rewritten
comments.

> <strike>This module name is obviously confusing</strike>
>
> <strike>What do you think about clarifying this module name?</strike>
>
> What do you think about clarifying the module name `UserCreator`? I found it a
  bit confusing since we're not actually creating a user. We're only inviting
  them. What do you think about something like `UserInvitation` or `UserInvite`?

After writing a comment, read it out loud to yourself. Is there possible
ambiguity? Can you be more specific, even if it's more verbose? Now
try to read it from the perspective of the author. Are there sentences that
sound angry, upset, or belittling?

Look at the example above. The first comment is potentially ambiguous (what
module is the reviewer talking about? Why is it confusing?) and belittling
("_obviously_ confusing"). The second comment is no longer belittling, but it's
still ambiguous (what's unclear about the current name?) The third comment is
clear (using a specific module name and stating why the reviewer finds it
confusing), it is not belittling (not "obviously confusing" but "_I_ found it
... confusing"), and it even anticipates that naming is difficult, so it makes a
few suggestions to help.

## Include code samples

If you're suggesting changes, don't speak in extremely vague terms and assume
your entire team understands what you mean. Instead, try to include code samples
of what you're recommending.

Here's an example of a comment I might write:

> What do you think about extracting the creation of the user so the whole
`#create` method is at the same level of abstraction? Maybe something like this?

>     # instead of this
>     def create
>       # this is at one level of abstraction
>       email = params[:email]
>       password = params[:password]
>       user = User.create(email, password)
>
>       # this is at a different level of abstraction
>       send_email_to_user(user)
>       notify_admin(user)
>     end
>
>     # we could have this
>     def create
>       # this is now all at the same level of abstraction
>       user = create_user
>       send_email_to_user(user)
>       notify_admin(user)
>     end
>
>     private
>
>     def create_user
>       email = params[:email]
>       password = params[:password]
>       User.create(email, password)
>     end

Of course, there are times when this may be unnecessary. For example, when
reviewing code from an author I know is familiar with what it means to write
methods at the same level of abstraction, I might simply ask the first question
above without including the code sample.

Whether or not the suggested change would benefit from a code sample will always
be, in the end, a matter of judgment. Nevertheless, consider that code reviews
are public conversations from which the whole team can benefit. And even
experienced developers gain from ideas on how to implement a suggested
change. So when in doubt, I recommend including the code sample.

## Find good things to say!

This one is very important. It's easy to list all the things you think need
changing in the pull-request but gloss over all the good things present. If you
see something good, say something good! It's refreshing to receive positive
feedback. I find that even simple things like these can go a long way:

* "I love this method extraction"
* "These tests look great! 🎉"
* "Nice catch on this poorly named method! Thanks for changing it"

## Include links, links, and more links

If you reference a function, don't assume the author of the code knows exactly
what function you're talking about. Include a link to its documentation.

> What do you think about using <strike>`map`</strike>
[`map`](https://ruby-doc.org/core-2.6.5/Enumerable.html#method-i-map) instead of
`each` here since we want the return value?

If you're referencing a previous commit or pull-request, include
a link to that.

> In <strike>a previous commit</strike>
> [172a0f](https://github.com/thoughtbot/ex_machina/commit/172a0f261af0711f5a84c7a9b44347e8aa84f21b),
> we made it easier to copy the dependency when installing.

And if you're talking about a particular concept with which the author may not
be familiar, try to include a link to a blog post that elaborates more on the
topic. Blog posts can be powerful tools for learning.

> What do you think about writing this acceptance test at the <strike>same level
> of abstraction</strike>
[same level of
abstraction](https://thoughtbot.com/blog/acceptance-tests-at-a-single-level-of-abstraction)?

Whatever you do, make it easy for the author to get to the things you reference.
Ideally they're only one click away.

*([Breaking down the fourth wall](https://en.wikipedia.org/wiki/Fourth_wall): did
you land here because someone linked this blog post in code review?)*

## Offer to chat and pair

Finally, I like to (a) offer to talk in person (or video call) to explain
anything that is unclear about my comments, and (b) volunteer to pair on any of
the changes I suggested. In addition to being a nice gesture, it opens the door
for further communication and collaboration. Just be sure to follow through when
someone takes you up on them.

## What next?

After writing this blog post, my co-worker (the great) Mike Burns shared this
[Reviewing Pull
Requests](https://chelseatroy.com/2019/12/18/reviewing-pull-requests/) blog post
with me. I found it very enjoyable, so if you liked this blog post, you might
enjoy it too!

And if you're interested in learning more about improving both sides of the code
review process, take a look at this talk on [Implementing a Strong Code-Review
Culture](https://www.youtube.com/watch?v=PJjmw9TRB7s).
