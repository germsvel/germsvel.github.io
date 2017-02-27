---
layout: post
title:  "An introduction to concurrency in Elixir"
categories: blog
tags: Elixir
---

Concurrency is a first-class citizen in the Elixir platform. The concurrency
model is that of isolated, independent processes that share no memory
and communicate via asynchronous message passing. In other words, a process
can run concurrently without concern of others, and processes communicate
with each other only via send-and-forget messages.

It's worth noting that these processes are not OS processes, they are BEAM processes.
They are lightweight, and you can have thousands of them running concurrently.

## Creating processes

In elixir we create new processes via,

[`spawn/1`][spawn/1], [`spawn/3`][spawn/3], [`spawn_link/1`][spawn_link/1] and [`spawn_link/3`][spawn_link/3]

each of which takes a function (anonymous or in a module) that will run in the new process.

* `spawn` creates a new process completely isolated from the one creating it.
* `spawn_link`, on the other hand, creates a link between the creating process and the created process.
We'll talk more about links further down below.

You can think of spawning a process as telling a process to run a function in isolation.
That function may call other functions (like an http endpoint), or it may call itself
infinitely in a loop (usually done to preserve state), or it may run the function and exit.

Let's open up `iex` and take a look at it. Note that the console is running in a
process itself. To see it's `pid`, type `self()`.


{% highlight elixir %}
iex(1)> self()
#PID<0.80.0>
{% endhighlight %}

Now, say we run the following in `iex`

{% highlight elixir %}
iex(2)> spawn(fn -> 1 + 1 end)
{% endhighlight %}

A new process is spawned, it runs the anonymous function `fn -> 1 + 1 end` and exits normally.
In `iex` we see the pid being returned but we do not see the result of `1 + 1` because
the processes are isolated, and the result is being returned in the spawned process.

But what if we want the spawned process to tell us the result? They
need to communicate via messages.

## Communication between processes

Elixir has simple primitives for message passing. They are [`send/2`][send/2] and `receive/1`.

We _send_ a message to a process, and each process _receives_ messages in their mailbox.

Let's now try to send the result of `1 + 1` in the example above to the creating process.

Open up `iex` and let's do the following,

{% highlight elixir %}
iex(1)> creator_pid = self()
iex(2)> spawn(fn ->
...(2)>   result = 1 + 1
...(2)>   send(creator_pid, {:result, result})
...(2)> end)
{% endhighlight %}

The code above spawns a new process with a function that calculates the result
of `1 + 1` and sends the result in a tuple to the creator pid (the `iex` process
in this case).

In order to receive the message, we use `receive/1` in `iex`.

{% highlight elixir %}
iex(3)> receive do
...(3)>   {:result, result} -> IO.inspect(result)
...(3)> end
2
{% endhighlight %}

With that, we should see the number `2` in our console. Communication between
messages achieved!

## Linking processes

Elixir processes are isolated, and that's great. But what do we do when two processes
are conceptually related to each other? What happens if we don't want process _x_ to
live if process _y_ dies? We _link_ them.

Links are an elixir primitive for detecting a process crash. These links are bi-directional,
meaning that when either process crashes, the other receives a notification that the
process has crashed. If this message is not handled, the receiving process will also
crash.

This allows us to propagate errors and take down groups of processes that should only
exist together. Now if we don't want process _x_ to exist without _y_, we can link them
and if one of them dies, it will also take down the other. We can do this via
[`Process.link/1`][process_link/1], [`spawn_link/1`][spawn_link/1], and [`spawn_link/3`][spawn_link/3]. Let's take a look at `spawn_link/1`
in `iex`.

We will spawn a linked process, and in
the function we will raise an error so that we cause the spawned process to crash. The
link should propagate the error to the creator process (the `iex` process) and
we should see the `iex` process go down. We will also see `iex` restart itself, but
that restarting mechanism is performed by a supervisor. It is not an automatic response of a
process that crashes.

In `iex`,

{% highlight elixir %}
iex(1)> spawn_link(fn ->
...(1)>   raise "what are you doing!"
...(1)> end)
{% endhighlight %}

And you should see something like this,

{% highlight shell %}
15:32:54.067 [error] Process #PID<0.109.0> raised an exception
** (RuntimeError) what are you doing!
  :erlang.apply/2
** (EXIT from #PID<0.80.0>) and exception was raised:
  ** (RuntimeError) what are you doing!
    :erlang.apply/2
{% endhighlight %}

where process 80 was the process running the `iex` console. Note that your
pids might be different from those in my example.

## Stopping error propagation

By default when a process receives an exit signal from another process (and the signal
is not `:normal`) the recipient is also taken down. That is why linked
processes go down when one exits with an error.

But what if we want to stop the propagation of errors? That is, what if we want a
process to be notified of the failure of another process but not be taken down with it?
For that we can _trap exits_.

### Trapping exits

Trap exit is a flag that can be set in a process. We can do so with
`Process.flag(:trap_exit, true)`.

When we set this flag in our process, it means that instead of receiving an exit signal
from its linked process, it will receive a message in its mailbox that the process has died.
This message is a tuple with of this form, `{:EXIT, from_pid, reason}`.

So if process _y_ dies, process _x_ now gets a message in its mailbox, and it can choose how it
wants to handle such a message.

Let's take a look at it in `iex`,

{% highlight elixir %}
iex(1)> Process.flag(:trap_exit, true)

iex(2)> spawn_link(fn ->
...(2)>   raise "what are you doing!"
...(2)> end)
{% endhighlight %}

At this point we should see that the spawned process crashed but the `iex` process was
not taken down like last time. Let's take a look at all the messages in the console with
the neat helper `flush/0`,

{% highlight elixir %}
iex(3)> flush()
{:EXIT, #PID<0.101.0>,
{% raw %} {%RuntimeError{message: "what are you doing!"},{% endraw %}
  [{:erlang, :apply, 2, []}]}}
{% endhighlight %}

## A note on supervisors

Supervisors are elixir processes that can restart other processes when they
fail. We saw this in action when the `iex` console died, and it was restarted
automatically. And though supervisors are out of the scope of this post, the
foundations of supervisors are all in here. A supervisor is, in essence, a
process that is linked to other processes, it traps exit signals, and restarts
the processes that crash.

## Further resources

I found the book [Elixir in Action](https://www.manning.com/books/elixir-in-action) extremely helpful
in understanding these concepts. I also recommend reading [learn you some erlang](http://learnyousomeerlang.com/).
It is great read.


[spawn/1]: https://hexdocs.pm/elixir/Kernel.html#spawn/1
[spawn/3]: https://hexdocs.pm/elixir/Kernel.html#spawn/3
[spawn_link/1]: https://hexdocs.pm/elixir/Kernel.html#spawn_link/1
[spawn_link/3]: https://hexdocs.pm/elixir/Kernel.html#spawn_link/3
[process_link/1]: https://hexdocs.pm/elixir/Process.html#link/1
[send/2]: https://hexdocs.pm/elixir/Kernel.html#send/2
