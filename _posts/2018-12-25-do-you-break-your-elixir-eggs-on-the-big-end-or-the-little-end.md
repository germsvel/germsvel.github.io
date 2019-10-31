---
layout: post
title: "Do you break your Elixir eggs on the big end or the little end?"
date: 2018-12-25
categories: blog
tags: elixir, endianess, functional programming
excerpt: >
  From big-endian to little-endian through unsigned integers. A tale of two ends,
  and Elixir shines again.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://robots.thoughtbot.com/do-you-break-your-elixir-eggs-on-the-big-end-or-the-little-end">thoughtbot.com/blog/do-you-break-your-elixir-eggs-on-the-big-end-or-the-little-end</a>.
</div>

> It is allowed on all hands, that the primitive way of breaking eggs, before we
> eat them, was upon the larger end; but his present majesty's grandfather,
> while he was a boy... happened to cut one of his fingers. Whereupon the
> emperor his father published an edict, commanding all his subjects, upon great
> penalties, to break the smaller end of their eggs. The people so highly
> resented this law, that our histories tell us, there have been six rebellions
> raised on that account; wherein one emperor lost his life, and another his
> crown.
> -- <cite>Gulliver's Travels</cite>

Most of the time when dealing with an Elixir app, you do not have to worry about
how your binaries are being represented (at least, I didn't have to). But
recently, while implementing the proof-of-work algorithm for Ethereum called
[Ethash], I found myself having to care a lot about the [endianess] of binaries.

I discovered that Elixir uses big-endian format by default. What that means in
practice is that when we write a binary as `<<4, 3, 2, 1>>`, we are assuming
that the bytes are ordered from most significant to least significant from left
to right. So `4` is the most significant byte, and `1` is the least significant
byte.

If we wanted to represent the same binary in little-endian format, we would
expect to see `<<1, 2, 3, 4>>`, where `1` is the least significant byte and `4`
is the most significant byte.

To better understand this, let's compare them by representing them as unsigned
integers,

```elixir
# big endian <<4, 3, 2, 1>> == little endian <<1, 2, 3, 4>>
iex> :binary.decode_unsigned(<<4, 3, 2, 1>>, :big)
67305985

iex> :binary.decode_unsigned(<<1, 2, 3, 4>>, :little)
67305985

# big endian <<1, 2, 3, 4>> == little endian <<4, 3, 2, 1>>
iex> :binary.decode_unsigned(<<1, 2, 3, 4>>, :big)
16909060

iex> :binary.decode_unsigned(<<4, 3, 2, 1>>, :little)
16909060
```

As you can see, those binaries can be decoded into the same unsigned integers.
They are just represented differently.

[endianess]: https://en.wikipedia.org/wiki/Endianness
[Ethash]: https://github.com/ethereum/wiki/wiki/Ethash

## Getting the least significant byte

Sometimes, we may only be interested in reading the least significant byte. When
our binaries are represented in big-endian format, we need to get the _last_
byte. But if we could represent our binary in little-endian format, we could
just get the _first_ byte. And that sounds easier.

How can we do this?

One way to deal with this is by turning our binary into a list, reversing that,
and grabbing the first element.

```elixir
<<4, 3, 2, 1>>
|> :binary.bin_to_list()
|> Enum.reverse()
|> hd()

# => 1
```

I don't like this very much because there's no indication that we're reversing
the binary to get the little-endian format.

A better way to deal with this is to decode the binary from big-endian into an
unsigned integer and then encode it as little-endian (essentially using the
equivalence we saw above),

```elixir
<<4, 3, 2, 1>>
|> :binary.decode_unsigned(:big) # could omit :big since it's default
|> :binary.encode_unsigned(:little)

# => <<1, 2, 3, 4>>
```

Then we can do some binary pattern matching to grab the first byte (8 bits),

```elixir
<<head::size(8), rest::binary>> =
  <<4, 3, 2, 1>>
  |> :binary.decode_unsigned(:big) # could omit :big since it's default
  |> :binary.encode_unsigned(:little)

head
# => 1
```

## List of unsigned integers

Other times, we may be interested in representing a large binary as a series of
unsigned integers (I had to do that a lot for the proof-of-work). Let's see how
we can do this assuming we want to turn a little-endian binary into a series of
32 bit unsigned integers.

If the binary in question is only four bytes long (32 bits), then the task is
simple. We could use `:binary.decode_unsigned/2`,

```elixir
:binary.decode_unsigned(<<1, 2, 3, 4>>, :little)
# => 67305985
```

If the binary is larger, and we do not know its size ahead of time, then we
could follow a brute-force approach by turning our binary into a list, grabbing
chunks of four, turning each of them back into binary, and decoding them.

Let's use `<<1, 2, 3, 4, 5, 6, 7, 8>>` as an example,

```elixir
<<1, 2, 3, 4, 5, 6, 7, 8>>
|> :binary.bin_to_list()
|> Enum.chunk_every(4)
|> Enum.map(&:binary.list_to_bin/1)
|> Enum.map(&:binary.decode_unsigned(&1, :little))

# => [67305985, 134678021]
```

That does the job, but there's a better way! Binary pattern matching allows us
to specify the [unit and size] of its parts, and it allows us to use [modifiers]
so that we can express exactly what we're trying to do.

Once again, let's start with a binary that is only four bytes long (32 bits).
Since we know the size of the binary, we can specify it via `size` and `unit`
options, and we can use the `unsigned` and `little` modifiers,

```elixir
# specific size and unit
<<number::size(4)-unit(8)-unsigned-integer-little>> = <<1, 2, 3, 4>>

number
# => 67305985

# or you can just use size * unit
<<number::size(32)-unsigned-integer-little>> = <<1, 2, 3, 4>>

number
# => 67305985
```

That last one reads particularly well (in my opinion) because it states exactly
what we want, _a 32 bit unsigned integer_ (from a little-endian binary).

And for larger binaries, we can combine what we know about binary pattern
matching with a [bitstring generator] for a really succinct result!

```elixir
binary = <<1, 2, 3, 4, 5, 6, 7, 8>>

for <<number::size(32)-unsigned-integer-little <- binary>> do
  number
end

# => [67305985, 134678021]
```

[unit and size]: https://hexdocs.pm/elixir/Kernel.SpecialForms.html#%3C%3C%3E%3E/1-unit-and-size
[modifiers]: https://hexdocs.pm/elixir/Kernel.SpecialForms.html#%3C%3C%3E%3E/1-modifiers
[bitstring generator]: https://elixir-lang.org/getting-started/comprehensions.html#bitstring-generators

## What's next?

I hope that by looking at these brief examples you are pleasantly surprised with
the excellent support Elixir has for dealing with binaries. I know I am!

If you found this interesting, I recommend reading the whole documentation
section for the [`<<args>>`
macro](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#%3C%3C%3E%3E/1).
There's some good stuff in there.

And last but not least... when dealing with endiannes, always be extra careful.
It can get confusing, and you might just reverse a few things.

Until next time!

```elixir
<<117, 110, 116, 105, 108, 32, 110, 101, 120, 116, 32, 116, 105, 109, 101>>
|> :binary.decode_unsigned()
|> :binary.encode_unsigned(:little)

"emit txen litnu"
```
