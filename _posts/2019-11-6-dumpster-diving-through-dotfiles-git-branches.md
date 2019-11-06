---
layout: post
title: "Dumpster Diving through Dotfiles: Remote Git Branches"
date: 2019-11-06
categories: blog
tags: git
excerpt: >
  Don't let your git repo become a graveyard of stale branches. But how do you
  even know what branches are there? Let me introduce you to `git branches`.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/dumpster-diving-through-dotfiles-git-branches">thoughtbot.com/blog/dumpster-diving-through-dotfiles-git-branches</a>.
</div>

In a previous blog post, I wrote about dumpster diving through dotfiles to get
[better git logging]. I wanted to dive once again into those depths to pull out
another very useful pearl.

![gif of Portlandia: woman in dumpster saying "This is perfectly good watermelon"](https://images.thoughtbot.com/blog-vellum-image-uploads/45qFs1s5TdGjdQS8vaw3_dumpster-diving.gif)

[better git logging]: http://www.germanvelasco.com/blog/2019/03/25/dumpster-diving-through-dotfiles-better-git-logging.html
[Ian Zabel]: https://twitter.com/iwz


Do you ever want to know which remote branches you have in your repo? I
frequently use `git branches` &mdash; a git alias from [thoughtbot's dotfiles]
&mdash; to do just that. It's something like `git branch` but for remote
branches. It's a superpower, and it comes as courtesy of the wonderful [Ian
Zabel].

Let's take a look at what it does in [FactoryBot]'s repo.

```shell
$ git branches
```

![output of running git branches](https://images.thoughtbot.com/blog-vellum-image-uploads/kYdXzU7BTnK2KNYzI4Zg_git-branches.png)

Wow! There are some very old branches out there. But aside from that, pretty
good, right?

[thoughtbot's dotfiles]: https://github.com/thoughtbot/dotfiles
[FactoryBot]: http://github.com/thoughtbot/factory_bot

## Diving in

If you look at the [gitconfig in thoughtbot's dotfiles], you'll find this alias:

[gitconfig in thoughtbot's dotfiles]: https://github.com/thoughtbot/dotfiles/blob/master/gitconfig

```shell
 branches = for-each-ref --sort=-committerdate --format=\"%(color:blue)%(authordate:relative)\t%(color:red)%(authorname)\t%(color:white)%(color:bold)%(refname:short)\" refs/remotes
```

It looks long and intimidating. So let's walk through it, piece by piece.

### git for-each-ref

The command we are running is `git for-each-ref`. The rest are options we are
passing to it. Let's first run it without options:

```shell
$ git for-each-ref
```

![output of running git for-each-ref](https://images.thoughtbot.com/blog-vellum-image-uploads/HXEylMSZTY22UH9f5F3Y_git-for-each-ref.png)

It looks like `git for-each-ref` is listing all the refs that we have.
Since we have not provided any options, it's listing all the branches
(`refs/heads`) that I have locally, all the remote ones (`refs/remotes`), and
any tags that the repo has (`refs/tags`).

Now that we have the basics, we can look at the options we're passing.
The description found in `git for-each-ref --help` says the following:

> Iterate over all refs that match `<pattern>` and show them according to the given
`<format>`, after sorting them according to the given set of `<key>`.

So we can pass a `<pattern>` to filter the list, a `<format>` to make the output
more intelligible, and a `<key>` to sort the results. Let's look at each of
those in turn.

### Filter with a `<pattern>`

Let's trim down our list of results to only show remote branches
(`refs/remotes`), since that's what our `git branches` alias does:

```shell
$ git for-each-ref refs/remotes
```

![output of running git for-each-ref refs/remotes](https://images.thoughtbot.com/blog-vellum-image-uploads/T98sxJEJS0KDXSnTZLDt_git-for-each-ref-refs-remotes.png)

Excellent. Now we're only listing `refs/remotes`. Let's add some formatting so
we can more easily understand what we're reading.

### Setting a `<format>`

The formatting option seems the most daunting when looking at the `git branches`
alias. But when we look at it more closely, it turns out it's not so bad. This
is the option in full.

```
--format=\"%(color:blue)%(authordate:relative)\t%(color:red)%(authorname)\t%(color:white)%(color:bold)%(refname:short)\"
```

It can be broken down into three main components:

* `%(color:blue)%(authordate:relative)`
* `%(color:red)%(authorname)`
* `%(color:white)%(color:bold)%(refname:short)`

So we want the relative date in blue, the author's name in red, and the branch's
name (the short version, without `refs/remotes/`) in bold white. We also
separate those three components with tabs (`\t`).

The `--format` option takes a string as an argument and interpolates the values we
saw above. So when running from the command line, we will not escape the opening
and closing quotes, and we will replace the tabs from `\t` to `%09` so they are
interpolated correctly. Let's run the following command:

```shell
$ git for-each-ref --format="%(color:blue)%(authordate:relative)%09%(color:red)%(authorname)%09%(color:white)%(color:bold)%(refname:short)" refs/remotes
```

![output of running git for-each-ref refs/remotes with --format option](https://images.thoughtbot.com/blog-vellum-image-uploads/wdaeNOj2TGWc9eTEXvmk_git-for-each-ref-formatted.png)

Nice formatting!

### Sorting with a `<key>`

Now for the last part. Let's sort them in descending order to see the most
recent first. We'll sort by `-committerdate` (note the `-` in front to denote
descending order).

```shell
$ git for-each-ref --sort=-committerdate --format="%(color:blue)%(authordate:relative)%09%(color:red)%(authorname)%09%(color:white)%(color:bold)%(refname:short)" refs/remotes
```

![output of running git for-each-ref refs/remotes with --format and --sort options](https://images.thoughtbot.com/blog-vellum-image-uploads/gQRi34nSSL6L9ybqeqJR_git-for-each-ref-sorted-full.png)

Excellent. We did it!

### Alias it

That is a long command to type. Before you go, make sure to alias it as `git
branches` or `git remote-branches` or something else to your liking.

```git
git config --global alias.branches 'for-each-ref --sort=-committerdate --format="%(color:blue)%(authordate:relative)%09%(color:red)%(authorname)%09%(color:white)%(color:bold)%(refname:short)" refs/remotes'
```

## Dumpster Dive through Dotfiles

If you enjoyed this, I encourage you to do some dumpster diving on your own to
find more treasure. The [thoughtbot dotfiles repo] is a great place to start.
There are more pearls to be found.

[thoughtbot dotfiles repo]: https://github.com/thoughtbot/dotfiles
[dumpster diving]: https://en.wikipedia.org/wiki/Dumpster_diving
[dotfiles]: https://thoughtbot.com/upcase/videos/intro-to-dotfiles
