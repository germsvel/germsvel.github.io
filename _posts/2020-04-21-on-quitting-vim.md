---
layout: post
title: "On Quitting Vim"
tags: vim
excerpt: >
  Vim has a reputation for being hard to quit. But it turns out there are so
  many ways to get out of it &mdash; it's like vim wants you to quit. Let's look
  at a few.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/on-quitting-vim">thoughtbot.com/blog/on-quitting-vim</a>.
</div>

![image of vim home screen](https://images.thoughtbot.com/blog-vellum-image-uploads/PjT9knnTAwwF3G1TLTdw_vim.png)

Vim has a reputation for being [hard to quit]. But it turns out there are plenty
of ways to do so! Recently, I decided to do a little digging and find out just
how many ways you can quit out of vim.

Here are a few (remember to always press `<Enter>` after the commands).

[hard to quit]: https://stackoverflow.blog/2017/05/23/stack-overflow-helping-one-million-developers-exit-vim/

## Just quit

A traditional way to quit vim is to type `:quit` or `:q` for short. That will
work for most people.

If there are changes you've made but don't care about, vim will still complain
and not let you quit. Silly vim. Just be more assertive --- `:quit!` or `:q!`.

![trying to quit with :q and then with :q!](https://images.thoughtbot.com/blog-vellum-image-uploads/liLKz7QyeglX6JZkJS5g_quit-and-bang-vim.gif)

## Save (write) and quit

What if you want to save your changes before quitting? And honestly, who
doesn't? Tell vim to write to the file (`w`) before quitting --- `:wq`.

You're good to go. Most people stop here and move on with their lives. But not
us. We have much to learn from vim yet.

## I want to type less!

What if I could save you 33% of the typing? Well, you can save and exit with
`:x`. You might say, how will I remember that? Think of `:xit` or `:exit` ---
both of which also work but are more typing.

## But I only want to save some of the lines I've changed

Wow, we're getting picky. Still, vim's got your back. You only want to save the
changes made to lines 1 through 3? Sure, just type `:1,3x!`.

## What if I want to save and exit from a different file?

You're trying to save and exit from a file in a different pane or window?
Aggressive, but not a problem. Just tell vim which file you want to save and
exit: `:x! path/to/other_file`.

## I just hate typing colons (`:`)

Hmmm. That's odd. But I suppose it's okay.

Vim still has you covered. You can quit without saving changes with `ZQ` or by
saving your changes with `ZZ`. It's like gamers saying
[GG](https://en.wikipedia.org/wiki/Glossary_of_video_game_terms#GG) at the end
of a game, except in this case, vim's the one winning. So `ZZ` -- `Zood Zame`?

## Can't vim just ask me nicely if I want to save files before quitting?

Sure. Sure it can. I know sometimes it's just nice to be asked. Just tell vim to
confirm with you before the changes: `:confirm quit` or `:conf q` for short.

![vim asking confirmation before quitting](https://images.thoughtbot.com/blog-vellum-image-uploads/M4NnRGO4Sj2Y8TbKvz7Z_vim-confirm-quit.png)

## Rage quit

What if we have several windows, more panes, and even more buffers open? I just
want to quit all vim! I understand. Just tell vim to `:quitall!`, `:qall!`, or
`:qa!` if you don't care about saving your changes. If you want to save your
changes, go for `:xall!` or `xa!`.

NOTE: `:xoxo` doesn't work -- vim doesn't want a hug or a kiss.

![quitting all vim panes](https://images.thoughtbot.com/blog-vellum-image-uploads/IULXlMr2SzqfJhDJergt_qa-vim.gif)

You can also ask vim to confirm each file before it closes it with `:conf xall`,
though that really takes the rage out of rage quitting.

## I need more help!

If you need more help, vim is there for you. Just type `:h :quit` and you'll see
all the possible options on how you can quit. Until next time!

`ZZ`
