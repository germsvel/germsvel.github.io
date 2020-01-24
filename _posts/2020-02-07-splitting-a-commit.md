---
layout: post
title:  "Splitting a Commit"
categories: git
tags: git
excerpt: >
  My workflow usually involves squashing many commits into a single one. But
  sometimes, the workflow calls for the opposite action -- splitting a single
  commit into many. This is how I do it.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/splitting-a-commit">thoughtbot.com/blog/splitting-a-commit</a>.
</div>

My workflow usually involves squashing many commits into a single one in
preparation for a pull request. But sometimes, I need to perform the opposite
action &mdash; splitting a single commit into many.

It may be that I realize one of my commits would better serve future developers
as two distinct commits in history (e.g. a refactoring commit that ["makes the
change easy"], and a feature commit that "makes the easy change"). Or perhaps, I
know I can unblock work for others by extracting some of the changes in the
commit into a separate pull request (e.g. several features need the same table).

Whatever the case, when I need to split a commit, I **rebase, edit, reset, and
commit**.

["makes the change easy"]: https://www.martinfowler.com/articles/preparatory-refactoring-example.html

## Rebase

Let's first do an [interactive rebase]. I like to keep my branch up to date with
the latest and greatest, so we'll go ahead and fetch and rebase from `master`:

```shell
$ git fetch origin
$ git rebase -i origin/master
```

We will be given a choice of what to do with our commits. In this case, we have
two of them:

```shell
pick da8f4d4 Adds greeting to application
pick 1c5e3b7 Adds greeting to jobs

# Rebase e42f496..1c5e3b7 onto e42f496 (2 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
```

Commit `1c5e3b7` ("Adds greeting to jobs") is fine as it is. But suppose we want
to split commit `da8f4d4` ("Adds greeting to application") to more closely match
the "jobs" commit &mdash; separating the changes in views, controllers, and
models to their own commits.

## Edit

In order to edit commit `da8f4d4`, we want the rebase to stop at that commit. So
change that commit's `pick` to `edit` (or just `e`):

```shell
edit da8f4d4 Adds greeting to application
pick 1c5e3b7 Adds greeting to jobs

# Rebase e42f496..1c5e3b7 onto e42f496 (2 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
```

Save and exit. We should now be in an editable state for commit `da8f4d4`:

```shell
Stopped at da8f4d4...  Adds greeting to application
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
```

## Reset

The output tells us about two options, neither of which are what we want:

❌ amend the commit message with `git commit --amend`, or

❌ continue with `git rebase --continue`

We don't want to simply amend the commit message &mdash; we want to change the
commit itself. And we don't want to continue with the rebase just yet. What we
want is a way to restart this commit as though we were trying to stage files for
the first time. And that's where secret option number three comes in:

✅ reset the code with `git reset HEAD^`and re-commit!

```shell
$ git reset HEAD^
Unstaged changes after reset:
M       app/controllers/application_controller.rb
M       app/models/application_record.rb
M       app/views/layouts/application.html.erb
```

Excellent! We now have all of our files unstaged, which we could confirm by
checking `git status`. Let's create the new commits next.

## Commit

We can now choose to add the changes in as many commits as we want. We could
even use `git add --patch` to select a subset of a file's changes. In this
exercise, we want to add the views, controllers, and models in separate commits:

```shell
# first commit
$ git add app/views
$ git ci -m 'adds greeting to views'
[detached HEAD 81c8f2b] adds greeting to views
 1 file changed, 1 insertion(+)

# second commit
$ git add app/controllers
$ git ci -m 'adds greeting to controllers'
[detached HEAD 3bf0765] adds greeting to controllers
 1 file changed, 5 insertions(+)

# third commit
$ git add app/models
$ git ci -m 'adds greeting to models'
[detached HEAD d77419c] adds greeting to models
 1 file changed, 4 insertions(+)
```

Now that we're satisfied with our new commits, we can continue with the rebase:

```shell
$ git rebase --continue
Successfully rebased and updated refs/heads/splitting-commit.
```

And voilà! We have split our commit into three, and now each commit has changes
for one section of the application:

```shell
$ git log --oneline --decorate -4
f5fd17f (HEAD -> splitting-commit) Adds greeting to jobs
d77419c adds greeting to models
3bf0765 adds greeting to controllers
81c8f2b adds greeting to views
```

## What next?

If you enjoy rewriting history and want to learn more about the many ways to do
so, take a look at the [Rewriting History docs], this [wonderful blog post], or
if you prefer videos, you can't go wrong with Chris Toomey's [Crafting History
with Rebase] video on Upcase.

[interactive rebase]: https://thoughtbot.com/blog/git-interactive-rebase-squash-amend-rewriting-history#interactive-rebase
[Rewriting History docs]: https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History
[wonderful blog post]: https://thoughtbot.com/blog/git-interactive-rebase-squash-amend-rewriting-history
[Crafting History with Rebase]: https://thoughtbot.com/upcase/videos/git-crafting-history
