---
layout: post
title:  "Splitting a Git Commit"
categories: git
tags: git
---

My workflow usually involves squashing many commits into a single one (and
adding a really good message of course!) But sometimes, the workflow calls for
the opposite action &mdash; splitting a single commit into many. When I need
that, this is how I do it.

## Rebase, edit, split, commit

I first do an interactive rebase. I like to keep my branch up to date with the
latest and greatest, so I'll go ahead and fetch and rebase from `master`.

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

Save that. We should now be in an editable state:

```
Stopped at c415f9e...  Adds greeting to application
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
```

The output tells us we can amend the commit message with `git commit --amend` or
we can continue with `git rebase --continue`, but it doesn't tell you about the
secret third option &mdash; to change the commit! Enter `git reset`:

```
$ git reset HEAD^
Unstaged changes after reset:
M       app/controllers/application_controller.rb
M       app/models/application_record.rb
M       app/views/layouts/application.html.erb
```

Excellent! We now have unstaged changes. If you type in `git status`, you should
see the very familiar look of unstaged files before you commit.

We can now choose to add files in as many commits as we want. Suppose (for the
sake of this exercise) that we want to add the views, controllers, and models
in separate commits.

```
# first commit
$ git add app/views
$ git commit -m 'add views'
[detached HEAD b5a0719] add views
 1 file changed, 1 insertion(+)

# second commit
$ git add app/controllers
$ git commit -m 'add controllers'
[detached HEAD bb7b50e] add controllers
 1 file changed, 5 insertions(+)

# third commit
$ git add app/models
$ git commit -m 'add models'
[detached HEAD ec6c1e7] add models
 1 file changed, 4 insertions(+)
```

Now that we're satisfied with our new commits, we can continue the rebase:

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

## You like to rewrite history?

If you like to rewrite history, take a look at the [Rewriting History] docs and
this [wonderful blog post].

[Rewriting History]: https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History
[wonderful blog post]: https://thoughtbot.com/blog/git-interactive-rebase-squash-amend-rewriting-history
