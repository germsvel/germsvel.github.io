---
layout: post
title:  "Splitting a Git Commit"
categories: git
tags: git
---

Usually my workflow involves squashing commits into a single one: I have a
feature branch, I commit as I hit stopping points, and before opening a
pull-request, I squash all commits into a single one (with a good message of
course!). But sometimes, the workflow calls for the opposite action &mdash;
splitting a single commit into many. When I need that, this is how I do it.

## Rebase, edit, split, commit

I first rebase my branch interactively. I like to keep my branch up to date with
the latest and greatest, so I'll go ahead and fetch and rebase from `master`
while I'm at it.

```
$ git fetch
$ git rebase -i origin/master
```

We will be given a choice of what to do with our commits.

```
pick c415f9e Adds greeting to application

# Rebase e42f496..c415f9e onto e42f496 (1 command)
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

In this case, we only
have one (`c415f93`), and we want to edit it. So change `pick` to `edit` (or
just `e`).

```
edit c415f9e Adds greeting to application

# Rebase e42f496..c415f9e onto e42f496 (1 command)
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

Saving that, we should now be in an editable state.

```
$ git rebase -i origin/master
Stopped at c415f9e...  Adds greeting to application
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
```

The output tells us we can amend the commit message with `--amend` or we can
continue with the rebase, but it doesn't tell you about the secret third option:
to change the commit!

```
$ git reset HEAD^
Unstaged changes after reset:
M       app/controllers/application_controller.rb
M       app/models/application_record.rb
M       app/views/layouts/application.html.erb
```

Finally! We now see that we have unstaged changes. If you type in `git status`,
you should see the very familiar look of unstaged files before you commit.

We can now choose to add however many commits we want. Suppose (for the sake of
this exercise) that you wanted to add the views, controllers, and models in
separate commits.

```
# first commit
$ git add app/views
$ git ci -m 'add views'
[detached HEAD b5a0719] add views
 1 file changed, 1 insertion(+)

# second commit
$ git add app/controllers
$ git ci -m 'add controllers'
[detached HEAD bb7b50e] add controllers
 1 file changed, 5 insertions(+)

# third commit
$ git add app/models
$ git ci -m 'add models'
[detached HEAD ec6c1e7] add models
 1 file changed, 4 insertions(+)
```

Now that we're satisfied with our new commits, we can continue with the rebase:

```
$ git rebase --continue
Successfully rebased and updated refs/heads/splitting-commit.
```
And voilà! You now have three commits instead of one.

```
$ git log --oneline --decorate -3
ec6c1e7 (HEAD -> splitting-commit) add models
bb7b50e add controllers
b5a0719 add views
```

## What next?

You like to rewrite history? Me too. Take a look at the [Rewriting History] docs.

[Rewriting History]: https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History
