---
layout: post
title: "Don't forget the silent step when you squash and merge"
date: 2018-12-24
categories: blog
tags: git
excerpt: >
  GitHub’s squash and merge button is great. But make sure to write a good commit
  message before you confirm those changes.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/don-t-forget-the-silent-step-when-you-squash-and-merge">thoughtbot.com/blog/don-t-forget-the-silent-step-when-you-squash-and-merge</a>.
</div>

I usually follow thoughtbot's [git protocol] when introducing new code into
`master`, where we rebase interactively (to rebase and squash commits) and then
we merge. But when I don't use that, I really like GitHub's [squash and merge]
button.

[git protocol]: https://github.com/thoughtbot/guides/blob/master/protocol/git/README.md
[squash and merge]: https://blog.github.com/2016-04-01-squash-your-commits/

![](https://images.thoughtbot.com/blog-vellum-image-uploads/pwn1Lx0Q8qi8qimVVohg_squash-and-merge.png)

But too often, I find that people who use that button tend to miss the middle
step (I might even call it _the silent step_) between squashing and merging.
They usually click "Squash and merge" and then immediately hit "Confirm squash
and merge",

![](https://images.thoughtbot.com/blog-vellum-image-uploads/rzYPjfGhSsypix38rwm5_confirm-squash-and-merge.png)

In doing so, they forget to write a good cohesive commit message! So what I'll
find is commit messages that look like this one,

    Author: Developer <dev@squashandmerge.com>
    Date: Thurs Dec 20 00:00:00 2018 --0400

      Add users to the checkout flow

      * wip: original implementation

      * add new routing for guests

      * hound updates

      * styling changes

      * fixed css nesting

      * change color for primary action

      * whoops, removed inspects

      * address feedback

      * hound again!

      * pass tests

The commit message above does not tell the reader _what_ changes were introduced
or _why_ they were introduced. It is simply a series of work-in-progress commits
that were done at a point in time. There might be a commit in there that
provides helpful information, but usually there is so much noise that it is hard
to find it. And the reader is left to guess at the reason behind them.

This is a crucial opportunity missed! With a good commit message, the author
could have communicated the reason behind the changes across space and time!

Wouldn't it be better if you found this?

    Author: Developer <dev@squashandmerge.com>
    Date: Thurs Dec 20 00:00:00 2018 --0400

      Add users to the checkout flow

      Trello: https://trello.com/z/aYaZaAAa/59-users-checkout-flow

      What changed?
      ======================

      This commit adds users to the checkout flow. In order to do that we had to
      add new routing for guests since our current implementation did not
      account for them.

      We also updated the colors of primary action buttons to better match the
      checkout flow.

      Why are we doing this?
      =====================

      We are introducing users to the checkout flow so that our customers can
      store items and come back to their shopping cart later. The actual work of
      persisting the shopping cart is not done here, but this lays the
      groundwork for all that future work.

## Write a cohesive commit message

So next time you hit the "Squash and merge" button, take a breather, and use the
text box conveniently located on top of the button to write a nice cohesive
commit message before you "Confirm squash and merge", changing something like
this

![](https://images.thoughtbot.com/blog-vellum-image-uploads/mZR7iHVSaE0wFGzT2DLg_confirm-squash-and-merge-bad-message.png)

into something like this,

![](https://images.thoughtbot.com/blog-vellum-image-uploads/1DEGsOaWRFezsgCCrG0Y_confirm-squash-and-merge-with-good-message.png)

That is much better. Now you're ready to "Confirm squash and merge"!
