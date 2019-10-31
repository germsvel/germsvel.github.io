---
layout: post
title: "Write good commit messages by blaming others"
date: 2018-09-03
categories: blog
tags: git, vim, tools, workflow, productivity
excerpt: >
  To write good commit messages, you must read commit messages. And to read commit
  messages, you must use `git blame` effectively.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/write-good-commit-messages-by-blaming-others">thoughtbot.com/blog/write-good-commit-messages-by-blaming-others</a>.
</div>

I mean `git blame`, not _actually_ blaming others.

## Valuing good commits

We are motivated to do good things because they are good. But we are extremely
motivated to do valuable things because they are valuable. Connecting the two
makes us extremely motivated to do good things.

This is true of writing good commit messages. We have been told that writing
good commit messages is good, and so it is! But unless we experience the value
of a good commit message, we will not be motivated to write them. And how do we
experience the value of a good commit message if we don't read commit messages?

To this end, [git-blame] allows us to discover which revision was the last to
change the file. But just knowing that is not good enough. We need to have a
workflow that makes "blaming" easy. The fewer steps we need to take from the
line of code we are trying to understand to the commit message the more likely
we are to actually read the message.

[git-blame]: https://git-scm.com/docs/git-blame

## Becoming fugitives

Vim has a great package called [vim-fugitive] that, among other commands, has a
`:Gblame` command. With the cursor on the line of interest, type `:Gblame`. This
will open the `git blame` output in a vertical split.

![opening git-blame vertical split with :Gblame](https://images.thoughtbot.com/blog-vellum-image-uploads/G6mYBbbHQneWgTtp9n1p_opening-git-blame-vertical-split-with-vim-fugitive.png)

But that's not all. With the cursor over the commit of interest, type `K`. This
opens up the commit message along with all the changes introduced in that
commit.

![pressing K on the commit sha reveals commit message](https://images.thoughtbot.com/blog-vellum-image-uploads/DOLmEwWiRA2k1hyBGc3A_pressing-k-reveals-commit-message.png)

That's just two steps, `:Gblame` and `K`. Compare that to the number of steps
required for the going-to-GitHub alternative:

1. look at the name of the file you are working on
2. open project in GitHub
3. look at the file name again because you forgot it
4. navigate to the file of interest in GitHub
5. click "Blame" button
6. go get cup of coffee
7. forget what you were doing
8. return to your computer
9. find side pane open
10. click on commit title on the left

That's 10 steps! (give or take a few)

Being able to quickly check the commit message for a line of code will make you
see the value in good commit messages. You will quickly see that `fix tests` is
not a valuable commit message to the reader. And you will also notice that
squashing down commits without actually writing a cohesive message wastes an
opportunity to provide valuable context for the future developer or designer who
will be looking at that code:

![example image of squash without editing the commit message](https://images.thoughtbot.com/blog-vellum-image-uploads/sWaASkQmCz9UWBVrwWw7_example-of-squash-without-cohesive-message.png)

We may have fewer commits, but the message hardly improved!

[vim-fugitive]: https://github.com/tpope/vim-fugitive

### Other editors

If you are not a Vim user, do not despair. A quick search shows that there are
packages for other editors that seem to do similar things (though not exactly in
the same way). I found some for [Emacs], [Atom], [VSCode], and [Sublime].

[Emacs]: https://magit.vc/manual/magit/Blaming.html
[Atom]: https://atom.io/packages/git-blame
[VSCode]: https://marketplace.visualstudio.com/items?itemName=waderyan.gitblame
[Sublime]: https://packagecontrol.io/packages/Git%20blame

## Okay, I see the value, but how do I write a good commit message?

Glad you asked! thoughtbot has blog posts that have really good advice on how to
write good messages. These are two that I find particularly helpful:

- [5 Useful tips for a better commit message]
- [Better commit messages with a .gitmessage template]

You can also check out thoughtbot's [gitmessage template] which provides a very
nice summary of what to include in a good message. I paste it here for your
convenience:

    # 50-character subject line
    #
    # 72-character wrapped longer description. This should answer:
    #
    # * Why was this change necessary?
    # * How does it address the problem?
    # * Are there any side effects?
    #
    # Include a link to the ticket, if any.
    #
    # Add co-authors if you worked on this code with others:
    #
    # Co-authored-by: Full Name <email@example.com>
    # Co-authored-by: Full Name <email@example.com>

[5 Useful tips for a better commit message]: https://thoughtbot.com/blog/5-useful-tips-for-a-better-commit-message
[Better commit messages with a .gitmessage template]: https://thoughtbot.com/blog/better-commit-messages-with-a-gitmessage-template
[gitmessage template]: https://github.com/thoughtbot/dotfiles/blob/master/gitmessage
