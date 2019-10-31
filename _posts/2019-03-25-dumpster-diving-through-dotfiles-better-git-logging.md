---
layout: post
title: "Dumpster Diving through Dotfiles: Better Git Logging"
date: 2019-03-25
categories: blog
tags: git
excerpt: >
  There are pearls to be found. Check out these git logs.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/dumpster-diving-through-dotfiles-better-git-logging">thoughtbot.com/blog/dumpster-diving-through-dotfiles-better-git-logging</a>.
</div>

A few months ago, I was pairing with the world-renowned command-line-tool expert
[Chris Toomey]. As is my wont, I tried to learn a trick or two from his workflow
to add to my repertoire. And that pairing session did not disappoint.

As we were pairing, it struck me how amazing his [git logging] was. It had
wonderful colors, there were different formats that he kept on using: at times
little graphs would show how different branches broke off at different points
from the `master` branch, while other times it was just a streamlined version of
commits that was exactly all we needed to see. Needless to say, I was blown
away. My `git log` looked bland by comparison, and more importantly, I didn't
have the kind of control that he had over how to present the log information.
All I saw was this:

[Chris Toomey]: https://twitter.com/christoomey
[git logging]: https://git-scm.com/docs/git-log

![git log output](https://images.thoughtbot.com/blog-vellum-image-uploads/24l48vXRFS3lLOZ7ygQe_plain-git-log.png)

So I did what any normal person would do and went dumpster diving through his
dotfiles.

![gif of dumpster diving](https://images.thoughtbot.com/blog-vellum-image-uploads/45qFs1s5TdGjdQS8vaw3_dumpster-diving.gif)

(By the way, if you're not familiar with dotfiles, take a look at this [intro
video] with Chris himself and Gabe -- also a command-line hero -- and if you're
not familiar with [dumpster diving], visit Brighton/Allston Massachussets on
September 1st, and you'll see it en masse.)

[dumpster diving]: https://en.wikipedia.org/wiki/Dumpster_diving
[intro video]: https://thoughtbot.com/upcase/videos/intro-to-dotfiles

I knew Chris was using some [git aliases], so I just went to his [gitconfig]
file and voila! I found the following entries under `aliases`:

```gitconfig
sl = log --oneline --decorate -20
sla = log --oneline --decorate --graph --all -20
slap = log --oneline --decorate --graph --all
```

[git aliases]: https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases
[gitconfig]: https://git-scm.com/docs/git-config

Let's ignore the aliases for now (we'll revisit them later), and let's focus on
decoding the options being passed to `git log`.

## git log --oneline --decorate -20

Let's first try `git log --oneline`

![git log --oneline output](https://images.thoughtbot.com/blog-vellum-image-uploads/vwhUw1A3QLiwrWGIeIg4_git-log--oneline.png)

The `--oneline` option is one of the most useful (in my opinion). It allows us
to see the commits in a more streamlined way, especially when we are not
interested in the full commit messages.

Let's now see what `git log --oneline --decorate` gives us:

![git log --oneline --decorate output](https://images.thoughtbot.com/blog-vellum-image-uploads/hjyZTgx9QYGYkv2K36i1_git-log--oneline--decorate.png)

As we can see, the `--decorate` option gives us some really helpful context for
tags and branch names (technically they are `ref/heads`, `ref/remotes`,
`ref/tags`, etc. which you can see with `--decorate=full`). Without them, I
would not have known that my `master` branch is at the same place as
`origin/master`. It's also very helpful for me to see tags like `v2.2.2` because
it allows me to know when we released that version of [ExMachina]!

[ExMachina]: https://github.com/thoughtbot/ex_machina

Okay, let's examine that last option, the `-20` part. Let's run `git log --oneline
--decorate -20`.

![git log --oneline --decorate -20 output](https://images.thoughtbot.com/blog-vellum-image-uploads/bLyIysRHKbAEFydPZBmA_git-log--oneline--decorate-20.png)

If you guessed the `-20` only shows you the most recent 20 commits, then you
were right!

Over the last few months, `git log --oneline --decorate -20` has become my
de-facto way of looking at `git log`.

## git log --oneline --decorate --graph --all -20

Let's now look at some of the other options we've seen with the alias `sla`,
namely `--graph` and `--all`.

Judging by the name, you might expect `--graph` to show us something graphical.
And indeed the `git-log` documentation says the following about the `--graph`
option:

> Draw a text-based graphical representation of the commit history on the left
> hand side of the output.

But just adding the `--graph` option didn't do much for me in terms of
usefulness. When I ran `git log --oneline --decorate --graph`, I had to scroll
down to some old commits to see some of the value:

![git log --oneline --decorate --graph output](https://images.thoughtbot.com/blog-vellum-image-uploads/ekYv0lcsQJ6LS9aFFELX_git-log--oneline--decorate--graph.png)

But what happens when we combine it with `--all`?

Running `git log --oneline --decorate --graph --all` brings on all the goods!

![git log --oneline --decorate --graph --all output](https://images.thoughtbot.com/blog-vellum-image-uploads/tBaXQ8HAREyhcZQIsjrk_git-log--oneline--decorate--graph--all.png)

By adding the `--all` option we can see all other branches and how they relate
to `master`. This is what I had seen Chris use that had blown my mind.

As you might guess, adding the `-20` option at the end will limit the number of
commits to 20, and I'd usually recommend using that option (otherwise you might
get a lot of output).

## Alias those commands to reduce typing

By this point, you may be wondering why Chris chose those aliases. I thought
about asking him, but by the time I had a chance, I had already labeled them in
my head, so I didn't want to ask and get them confused. This is how I remember
mine:

* `git sl`: I think of "git single line".
* `git sla`: I think of "git single line all"
* `git slap`: When you have a lot of branches in a project, the volume of
  information that comes from this command is like a slap in the face. So I only
  use this alias for cases when I want to get "git slapped".

I trust that after seeing my "git slapped" mnemonic device, you will prefer my
memorization techniques to whatever Chris may have had in mind. But in case you
didn't, I leave as an exercise for you to ask him in twitter.

Finally, if you like these (or any other combination of them) be sure to add
them to your git config because it is a lot of typing otherwise:

```bash
$ git config --global alias.sl 'log --oneline --decorate -20'
$ git config --global alias.sla 'log --oneline --decorate --graph --all -20'
$ git config --global alias.slap 'log --oneline --decorate --graph --all'
```

## Dumpster Dive through Dotfiles

Even if you didn't like any of the logging options I presented here, I still
encourage you to do some dumpster diving on your own. The [thoughtbot dotfiles
repo] is a great place to start. There are pearls to be found.

[thoughtbot dotfiles repo]: https://github.com/thoughtbot/dotfiles

![gif of someone eating fruit from a dumpster](https://images.thoughtbot.com/blog-vellum-image-uploads/35CIaycS2ufkWyjhCCjS_dumpster-diving-final.gif)
