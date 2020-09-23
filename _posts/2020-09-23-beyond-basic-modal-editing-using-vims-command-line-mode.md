---
layout: post
title: "Beyond basic modal editing. Using vim's command-line mode."
tags: vim
excerpt: >
  Most people learn vim's `normal`, `insert`, and `visual` modes. But they're
  only casually acquainted with vim's powerful Ex commands. Let's take a look at
  some.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/beyond-basic-modal-editing-using-vims-command-line-mode">thoughtbot.com/blog/beyond-basic-modal-editing-using-vims-command-line-mode</a>.
</div>

Vim is famous for its fabulous modal editing. Its `normal`, `insert`, and
`visual` modes quickly become magic in the hands of an able user. But many are
unaware or ignore the existence of vim's `command-line` mode with its Ex
commands.

Whereas vim's `normal` and `insert` mode work in the locale of your cursor,
vim's `command-line` mode can help you deal with things far from your cursor. Or
as Drew Neil puts it in his [Practical Vim] book,

> Vim's Ex commands strike far and wide.

Let's look at a few of them.

## Copy (copy, t), move (m), and delete (d)

We'll start by copying, moving, and deleting lines. You access vim's
`command-line` mode with `:`. Copying, moving, and deleting can be intuitive.
What would you say this command did?

```
:3copy5
```

If you guessed copy line three to line five, then you got it!

![Copying line 3 to 5 with `:3copy5`](https://images.thoughtbot.com/blog-vellum-image-uploads/RGJOfCyZQIyBKosuisZU_ex-copy.gif)

`t` is shorthand for copy (think copy _to_). So `:3t5` does the same thing as
the command above.

Do move and delete work the same way? You betcha.

Try moving a line with `:3m5`.

![Moving line 3 to 5 with `:3m5`](https://images.thoughtbot.com/blog-vellum-image-uploads/UOOh429tSFenaQYI8NC3_ex-move.gif)

Now try deleting a line with `:3d`.

![Deleting line 3 with `:3d`](https://images.thoughtbot.com/blog-vellum-image-uploads/zg6ARlOvSZKDWxHE6B0T_ex-delete.gif)

## Acting on ranges

But that's not all. Ex commands aren't limited to single lines. They can act on
ranges. Want to move lines 3-5 to line 8? You got it:

```
:3,5m8
```

![Moving lines 3-5 to 8 with `:3,5m8`](https://images.thoughtbot.com/blog-vellum-image-uploads/JNdmEtYTWOHQ2qe2QvHQ_ex-move-range.gif)

### Relative ranges

I know what you're thinking, "But what if I'm on line 125, and I use _relative_
numbers!" Not to worry. Vim has a range for you too:

```
:-2,+1d
```

![Delete the two previous line and the next line with `:-2,+1d`](https://images.thoughtbot.com/blog-vellum-image-uploads/y3lEMU4KSj2Wj3u5Died_ex-relative-range.gif)

### The visual range

What about using a visual selection as a range? If you're like me, you've
probably used vim to help you sort lines by visually selecting them and typing
`:sort`. Did you ever notice the range vim inserts for us? It's the special
visual selector range:

```
:'<,'>sort
```

![Visually select lines 3-7 and sort with `:sort`](https://images.thoughtbot.com/blog-vellum-image-uploads/SZJUq2czTD1fwfaNPjBE_ex-sort-visual-range.gif)

Want to perform other actions on that same visual block? The visual selector
range (`'<,'>`) continues to operate on the previous visual selection, even when
that range is no longer selected! Let's delete the lines we sorted last time:

```
:'<,'>d
```

![Delete lines previously visually selected with `:'<,'>d`](https://images.thoughtbot.com/blog-vellum-image-uploads/wmjOC0Y5SW64sfN5NxUp_ex-visual-range-delimiters.gif)

Want to learn more about the visual selector range? As it turns out, ranges can
be delimited by [marks], and `'<` and `'>` are just a couple of [special marks]. That opens
ranges to a world of possibilities! Try `'{,'}` for paragraphs and `'(,')` for
sentences.

[marks]: http://vimdoc.sourceforge.net/htmldoc/motion.html#mark-motions
[special marks]: http://vimdoc.sourceforge.net/htmldoc/motion.html#'%3C

## Search and replace

Perhaps you've never stopped to think about the magical incantation you use to
search and replace words in a file. Well, we can finally uncover the mystery:
`s` will replace things (think _substitute_). Want to replace `foo` with `bar`
on line 3?

```
:3s/foo/bar/g
```

![Replace foo with bar on line 3 with `:3s/foo/bar/g`](https://images.thoughtbot.com/blog-vellum-image-uploads/cuDeifTrCo6KI2LoHjrQ_ex-sub-one-line.gif)

What if you want to search and replace in the entire file? We can use a range
from the first line to the end of the file: `1,$`, or even better, we can use
vim's shorthand for the whole file: `%`. Go ahead. Type it in, and see the
magic work:

```
:%s/foo/bar/g
```

![Replace foo with bar on all lines with `:%s/foo/bar/g`](https://images.thoughtbot.com/blog-vellum-image-uploads/ZS0syn4RoS4ziVHcxRrw_ex-sub-whole-file.gif)

## Repeat a normal command

In addition to modal editing, vim is known for its ability to repeat the last
command with `.`. Maybe you delete a line with `dd`, then you realize you need
to delete the next one. No need to press `dd` again, just type `.` and you're
good to go.

But what about repeating the same command across many lines? What would you do
if you wanted to append a comma at the end of each line in a list? Ex commands
are here to help you:

```
:2,5normal A,
```

![Append comma on lines 2-5 with `:2,5normal A,`](https://images.thoughtbot.com/blog-vellum-image-uploads/awV79faXTz6EyXKpAGm1_ex-repeat-normal-command.gif)

## Repeat the last Ex command

`.` repeats the last normal command. What if we want to repeat the last Ex
command? We can do that too.

```
@:
```

![Repeat last command with `@:`](https://images.thoughtbot.com/blog-vellum-image-uploads/Dkvo8PnsTTG6k4IOaRgu_ex-repeat-last-ex-command.gif)

If you've ever used macros in vim, you might notice this looks like a macro for
the `:` register. Find that interesting? Take a look at your registers (`:reg`)
and see what you find for `:`.

## What next?

I hope you see the power of Ex commands. Instead of typing `2Gdd` to go to the
second line and delete it, you can delete it from afar with `:2d`. And instead
of going to line 3, visually selecting lines 3-5, yanking them, going down to
line 10, and pasting the lines with `3GVjjjy10Gp`, you can just do it from
anywhere in the file with `:3,5t10`.

If you want to learn more, I highly recommend Drew Neil's [Practical Vim] book,
or take a look at the excellent documentation for different [vim modes] and for
the Ex commands [copy], [move], [delete], [substitute]. And if you want to dig
deeper, help is just a `:help` away.

Until next time!

[Practical Vim]: https://pragprog.com/titles/dnvim2/practical-vim-second-edition/
[copy]: http://vimdoc.sourceforge.net/htmldoc/change.html#:copy
[move]: http://vimdoc.sourceforge.net/htmldoc/change.html#:move
[delete]: http://vimdoc.sourceforge.net/htmldoc/change.html#:d
[substitute]: http://vimdoc.sourceforge.net/htmldoc/change.html#:s
[vim modes]: http://vimdoc.sourceforge.net/htmldoc/intro.html#vim-modes
