---
layout: post
title: "Breaking Out of Ecto Schemas"
categories: ecto, elixir, sql, web
tags: ecto, elixir, sql, web
excerpt: >
  With Ecto, 🎶 you _can_ always get what you want. And if you try sometimes,
  well, you might find, you can _select_ want you need 🎶.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/breaking-out-of-ecto-schemas">thoughtbot.com/blog/breaking-out-of-ecto-schemas</a>.
</div>

Typically, when writing queries with Ecto, we use a module that uses an
[Ecto.Schema]. By default, those queries return all fields defined in that
schema. That makes a lot of sense when the intention is to retrieve a fully
populated struct from the database.

But sometimes, we only want a subset of the fields defined in the schema. In
fact, we may not even want to use a schema at all! For those cases, Ecto gives
us the ability to drop one step closer to the raw power of SQL. Let's take a
look at an example.

[Ecto.Schema]: https://hexdocs.pm/ecto/Ecto.Schema.html

## Getting active users with comments

Suppose we want a report with a list of active users along with how many
comments they have made. Instead of using an ecto schema for the query, we can
specify the table directly. And instead of preloading all comments to count them
in Elixir, we can use our good old SQL friends [`SELECT`,`COUNT`, and `GROUP
BY`](https://thoughtbot.com/blog/back-to-basics-sql#group-by):

```elixir
query =
  from u in "users",
  join: c in "comments",
  on: c.user_id == u.id,
  where: u.active == true,
  group_by: [u.name, u.email],
  select: [u.name, u.email, count(c.id)]

Repo.all(query)

# => [
    ["Gandalf", "gandalf@greypilgrim.com", 23],
    ["Aragorn", "strider@rangers.com", 45],
    ["Gimli", "sonofgloin@lonelymountain.com", 566],
    [...],
    ...
  ]
```

Great! By using `select`, we don't fetch unnecessary data to populate the full
`%User{}` and `%Comment{}` structs, and by using `count`, we avoid having to
preload all user comments just to count them in Elixir.

But not all is sunshine and rainbows yet. The return data is a list of lists,
which means we can't pass around that data without also having to specify that
the first element of each list is the name, the second is the email, and the
third is the number of comments for that user. And what if we were to return
four, five, or six columns? It gets out of hand very quickly.

Fortunately, Ecto has a better way!

### Mapping your select

Ecto lets us define the structure in which to return the data. For this report
we can define a descriptive map:

```elixir
query =
  from u in "users",
  join: c in "comments",
  on: c.user_id == u.id,
  where: u.active == true,
  group_by: [u.name, u.email],
  select: %{name: u.name, email: u.email, comments_count: count(c.id)}

Repo.all(query)

# => [
    %{comments_count: 23, email: "gandalf@greypilgrim.com", name: "Gandalf"},
    %{comments_count: 45, email: "aragorn@rangers.com", name: "Aragorn"},
    %{comments_count: 566, email: "sonofgloin@lonelymountain.com", name: "Gimli"},
    %{...},
    ...
  ]
```

That's so much better! We now have the best of both worlds --- a lean query that
gets only what we want in a structure we can pass around without having to
provide additional context.

## What next?

If you liked this, take a look at [Programming Ecto]. Even though I've been
using Ecto for quite some time, I recently read it and learned quite a few
things. I highly recommend it.

[Programming Ecto]: https://pragprog.com/book/wmecto/programming-ecto
