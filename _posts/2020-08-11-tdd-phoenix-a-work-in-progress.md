---
layout: post
title: "TDD Phoenix &mdash; update on a work in progress"
date: 2020-08-11
categories: blog
tags: tdd, bdd, phoenix, elixir, book, tutorial
excerpt: >
  I've been writing a <a href="https://www.tddphoenix.com">TDD Phoenix</a> book.
  It's still a work in progress, but it's freely available online. This is a
  short update on progress and plans.
---

At the end of June, I
[tweeted](https://twitter.com/germsvel/status/1276531321922367488) that I was
writing a [Test-Driven Development with
Phoenix](https://www.tddphoenix.com/) book. It's still a work in progress, but I
wanted to share it with the community for several reasons. I wrote a little bit
about why in [TDD Phoenix - a work in progress](https://www.tddphoenix.com/work-in-progress).


## Progress

Since the announcement, I have reviewed most of the text in the book and
improved it. But I'm not done. The last two chapters still need work, and I need
to write a _Conclusion_ chapter.

I have also gone over some parts of the app to simplify it. In particular, I
removed a lot of convoluted refactoring in [Putting the Chat in Chat
Rooms](https://www.tddphoenix.com/putting-the-chat-in-chat-rooms/). A huge
thanks to [Chris Turner](https://twitter.com/slowburnaz) for pointing out some
issues in there.

Finally, I updated the site to use Tailwind CSS's [typography
plugin](https://tailwindcss.com/docs/typography-plugin/#app) and updated the
syntax highlighting. I'm pleased with the result, though of course, I want to do
more. But it's good enough for now. I tweeted some
[before](https://twitter.com/germsvel/status/1285190947421261824) and
[after](https://twitter.com/germsvel/status/1288597842332975105) screenshots,
and here's how it looks now:

---

![Sample of typography in TDD Phoenix](https://pbs.twimg.com/media/EdXp6L1WkAAxqVU?format=png)

---

![Sample syntax highlighting from TDD
Phoenix](https://pbs.twimg.com/media/EeIEl8FWAAAJcRl?format=png)

---

## Plan

My plan is to finish editing the book's text, then go through every code sample
to rebuild the app step by step, and then edit the book again. Once that's done,
I'd like to update it to the latest version of Phoenix and Wallaby.

I already have several code improvements I want to make, but I need to be
careful not to remove all complexity. It's difficult to explain complex
refactors without losing the reader, but I want to keep some of those refactors
in the book since that's real application-building. If I completely smooth out
all the edges, there will be no need for refactoring, and TDD will become
red-green &mdash; no refactoring.

## Community Acceptance & Feedback

I was terrified to publish a book, led alone one that is a work in progress. I
hinted at that in the [work in
progress](https://www.tddphoenix.com/work-in-progress) announcement as one of
the reasons why I wanted to share the unfinished book:

>Because it is always scary to write something and share it with the world. So I want
>to put it "out there" to be rid of that fear, lest the fear wins, and I fail to
>share it.

It's never easy to share something that others can criticize. But because I knew
I was afraid, I went ahead and did it. It worked out okay.

1. It's out there. No need to hide it. And it's a relief to be able to work in
   the open.

2. The community response was great! People were very encouraging. It was very
   nice to see [Elixir School's
   response](https://twitter.com/elixirschool/status/1276623920347504640) to my
   announcement and to be mentioned in the news section of the [Thinking Elixir](
   https://thinkingelixir.com/podcast-episodes/003-elixir-1-11-preview-with-wojtek-mach/)
   podcast.

3. And best of all, people actually read the book! They sent me direct messages
   via twitter with suggestions or corrections. What a great feeling.

In the future, I hope to put updates via email &mdash; there's a sign up form at
the bottom of [TDD Phoenix](https://www.tddphoenix.com) now &mdash; and then
posting them here. Hope you follow along in whatever way works best for you!
