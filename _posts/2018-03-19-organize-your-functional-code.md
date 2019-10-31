---
layout: post
title: "How to organize your functional code"
date: 2018-03-19
categories: blog
tags: elixir functional-programming good-code
excerpt: >
  When coming from object-oriented languages, I often hear people ask the
  question, “How do I organize my code? Modules are just bags of functions!”. That
  is a question I asked myself as well, but after using Elixir and Elm for a
  while, I have noticed that there is a principle of organization that I keep
  using and that I see in the wild. I like to think of it as the principle of
  attraction.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/organize-your-functional-code">thoughtbot.com/blog/organize-your-functional-code</a>.
</div>

When coming from object-oriented languages, I often hear people ask the
question, "How do I organize my code? Modules are just bags of functions!". That
is a question I asked myself as well, but after using [Elixir] and [Elm] for a
while, I have noticed that there is a principle of organization that I keep
using and that I see in the wild. I like to think of it as the principle of
attraction.

## The principle of attraction

> Let data attract behavior.

I don't suppose I invented this, but the intent behind the principle of
attraction is that you should organize your modules and functions (behavior)
around data structures and abstractions (data). In Elixir, the principle would
tell us to organize our modules and functions around Elixir [structs],
[behaviours], and [protocols].

Let's take a look at an example of a popular Elixir library, [Plug]. `Plug` is a
great library for building web applications. It handles anything (and
everything) related to the request and response. If you come from the Ruby
world, `Plug` will remind you of [Rack].

At the core of `Plug` are two things, a connection struct called [Plug.Conn] and
a specification for what exactly is a plug. Once you notice those two things,
you will see that all modules and functions revolve around one of those two
things.

[Plug]: https://hexdocs.pm/plug/readme.html
[structs]: https://elixir-lang.org/getting-started/structs.html
[behaviours]: https://elixir-lang.org/getting-started/typespecs-and-behaviours.html
[protocols]: https://elixir-lang.org/getting-started/protocols.html
[Elixir]: https://elixir-lang.org/
[Elm]: http://elm-lang.org/
[Rack]: https://rack.github.io/

## Plug.Conn

First, note that all functions within the [Plug.Conn] module are related to a
connection struct, which is defined inside that very module. But if you take a
closer look, you will also see _how_ each of those functions takes in a
connection struct.

Here's a list of a few functions defined in [Plug.Conn],

* `assign(conn, key, value)`
* `clear_session(conn)`
* `put_resp_content_type(conn, content_type, charset)`
* `put_resp_header(conn, key, value)`
* `put_session(conn, key, value)`
* `send_resp(conn, status, body)`

Notice a pattern? They all take `conn`, the connection struct, as their first
argument, and though you can't see it, they all return a (modified) connection
struct.

They do that because this module revolves around the `Plug.Conn` struct. And
passing the connection struct as the first argument allows for composition via
pipelines.

Take a look at this example for interacting with a request,

```elixir
conn
|> put_session(:current_user, user)
|> put_resp_header(location, "/")
|> put_resp_content_type("text/html")
|> send_resp(302, "You are being redirected")
```

Very clean, right?

So, the first way to apply the principle of attraction is to organize your
modules and functions _around_ the struct that they are modifying. The struct
attracts the functions.

[Plug.Conn]: https://hexdocs.pm/plug/Plug.Conn.html

## Plug as a specification

The second way in which Plug is organized is around the notion of
_a_ plug. That may sound like a tautology, but what I mean is that many `Plug`
modules make use of a plug or are themselves valid plugs (that is they follow
the `Plug` specification).

What is the `Plug` specification? Glad you asked.

In order for a module to be a valid plug, it must define an `init/1` function
that takes a set of options and returns a set of options, and it must define a
`call/2` function that takes a connection struct as its first argument, the set
of options as its second one, and it must return a connection struct. A function
plug simply has to abide by the same specification as the `call/2` function in
the module plug, taking the connection struct and the options as arguments, and
returning a connection struct.

This is an example of a valid module plug,

```elixir
defmodule CustomPlug do
  def init(opts) do
    opts
  end

  def call(conn, _opts) do
    conn
  end
end
```

Now let's take a look at how `Plug`, the library, builds modules around the
notion of plugs.

[Plug.Builder] is a module in the `Plug` library that allows us to define a
pipeline of plugs that will be executed sequentially in the order in which they
are defined. The magic really comes in when many modules in the `Plug` library
are plugs themselves, and can be thus used in such a pipeline.

Let's look at an example,

```elixir
defmodule MyPlugPipeline do
  use Plug.Builder

  plug Plug.Logger
  plug Plug.RequestId
  plug Plug.Head
  plug :hello

  def hello(conn, _) do
    Plug.Conn.send_resp(conn, 200, "Hello world!")
  end
end
```

Note how [Plug.Logger], [Plug.RequestId], and [Plug.Head] are themselves plugs
and for that reason can be used in the pipeline provided by `Plug.Builder`. We
can also mix and match by defining our own function plug within the module (the
`plug :hello` part).

By organizing modules and functions around the plug abstraction, `Plug.Builder`
allows us to compose rich pipelines with other modules and functions that
satisfy the abstraction.

[Plug.Builder]: https://hexdocs.pm/plug/Plug.Builder.html
[Plug.Logger]: https://hexdocs.pm/plug/Plug.Logger.html
[Plug.Head]: https://hexdocs.pm/plug/Plug.Head.html
[Plug.RequestId]: https://hexdocs.pm/plug/Plug.RequestId.html

## What next?

I certainly think there are other principles by which to organize your
functional code, but I do recommend keeping this one handy. I have found it to
be quite useful! It even works when modules are nested within other modules. For
example, [Plug.Conn] does not have all the logic related to a connection struct
in itself. Sometimes it uses other modules that are namespaced under `Plug.Conn`
like [Plug.Conn.Status]. But if you look closely, `Plug.Conn.Status` also uses
the principle of attraction by having all of its functions deal with the same
piece of data, a status map defined inside the module.

[Plug.Conn.Status]: https://hexdocs.pm/plug/Plug.Conn.Status.html
