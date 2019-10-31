---
layout: post
title: "Long-lived processes in Elixir"
date: 2018-06-18
categories: blog
tags: elixir, functional-programming, processes, concurrency
excerpt: >
  Long-lived processes are everywhere in Elixir. Let’s look at why they’re needed
  and how to create them!
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/long-lived-processes-in-elixir">thoughtbot.com/blog/long-lived-processes-in-elixir</a>
</div>

One of the things that I loved about Elixir when I first started using the
language was the fact that everything runs inside a
[process](https://elixir-lang.org/getting-started/processes.html) (an Erlang VM
process, not an operating system process). Each process is lightweight and
isolated, and creating and destroying them is fast. As such, Elixir's processes
are front and center, which makes interacting with them both necessary and
wonderful.

For example, if you start the interactive console, `iex`, you'll see that all
your commands are running in a process:

```sh
$ iex
Interactive Elixir

iex> self()
#PID<0.84.0>

iex> 1 + 1
2

iex> self()
#PID<0.84.0>
```

so all of the code you run in the console is running in process `84` (an
Elixir/Erlang process).

Naturally, you can choose to run the code in a separate process,

```sh
iex> spawn(fn -> 1 + 1 end)
#PID<0.86.0>
```

`spawn/1` creates a new process which runs the function provided, `fn -> 1 + 1
end`. Interestingly, we do not see the return value of the anonymous function
because the function ran in a different process. What we get instead is the
process id, `pid`, of the spawned process.

Notice moreover that I said that the anonymous function _ran_ in another process
— ran as in past tense. Once process `86` finished doing all of its work, it
exited and we can no longer get any data from it.

```sh
iex> pid = spawn(fn -> 1 + 1 end)
#PID<0.86.0>

iex> Process.alive?(pid)
false
```

So if a process exits when it is finished with its work, how can we have a
long-lived process in Elixir?

Well... we just need to give the process work that never ends.

## A never-ending story

How can we give a process work that never ends?

As a developer you may have experienced (accidentally) creating an infinitely
recursive loop in a program, causing the program to hang for a while and
eventually crash. In our case we want to do this intentionally.

Let's see it in action by writing a simple loop,

```elixir
defmodule LiveLong do
  def run do
    run()
  end
end
```

That's the simplest loop I can think of. Now in `iex` we can spawn a process
that uses that function,

```sh
iex> pid = spawn(LiveLong, :run, [])
#PID<0.96.0>

iex> Process.alive?(pid)
true

iex> Process.info(pid)
[
  current_function: {LiveLong, :run, 0},
  status: :running,
  # more info
]
```

There we go! We have created a long-lived process. It will keep running in an
infinite loop until we tell it to exit. But magically, it will not crash our
program.

### Why doesn't it crash?

In other languages, if you get stuck in an infinitely recursive loop, your program
hangs for a while and then crashes. Why doesn't that happen here?

Elixir has [tail call] optimization (or really [last call optimization]). That
means that so long as we call the same function at the end of the execution
path, Elixir will not allocate a new stack frame to the call stack, and our
program will not suffer from a [stack overflow].

[tail call]: https://en.wikipedia.org/wiki/Tail_call
[last call optimization]: https://elixirforum.com/t/endless-recursion/10683/6
[stack overflow]: https://en.wikipedia.org/wiki/Stack_overflow

## Okay, but why do I care?

It is true that you may not need a long-running process if all you need is to
run a script or perform a one-off task. But having long-running processes is key
to programming in Elixir. For example, it is how [GenServers] can send and
receive messages while keeping state, and it is how [Supervisors] can provide
fault tolerance to your system by knowing and restarting other processes that
crash.

In order to make this sink in even more, let's create something that resembles a
simple `GenServer` - a process that is long-lived, can handle messages, and can
store state.

```elixir
defmodule SimpleGenServer do
  def start do
    initial_state = []
    receive_messages(initial_state)
  end

  def receive_messages(state) do
    receive do
      msg ->
        {:ok, new_state} = handle_message(msg, state)
        receive_messages(new_state)
    end
  end

  def handle_message({:store, value}, state) do
    {:ok, [value | state]}
  end

  def handle_message({:send_all_values, pid}, state) do
    send(pid, {:all_values, state})
    {:ok, state}
  end
end
```

Let's do a quick breakdown of each function,

- `start/0` simply defines an empty list as the initial state and calls
  `receive_messages/1`.

- `receive_messages/1` is where the magic happens. Here we use the `receive/1`
  Elixir primitive to pattern match messages out of our mailbox. In our case, we
  grab any message that comes in (`msg`) and pass it to our `handle_message/2`
  function along with the `state`. The `handle_message/2` function will do
  something with the message and state, and we expect it to return an updated
  state. We then perform the magic trick of making a recursive tail call. We
  call `receive_messages/1` with the new state.

- `handle_message({:store, value}, state)` will simply pattern match when we
  want to store a value, and it will add it to the current state.

- `handle_message({:send_all_values, pid}, state)` receives a `pid` to which it
  will send all the values found in the process (the current state).

Now let's interact with it,

```sh
iex> pid = spawn(SimpleGenServer, :start, [])
#PID<0.146.0>

iex> send pid, {:store, 23}
iex> send pid, {:store, "hello"}

iex> iex_pid = self()
iex> send pid, {:send_all_values, iex_pid}

iex> flush()
{:all_values, ["hello", 23]}
```

where `flush/0` is a convenience function in `iex` that allows us to get all the
messages in the console's mailbox instead of having to pattern match them out
with `receive/1`.

As you can see, our long-lived process successfully stored the data we sent to
it, and we were able to retrieve all the values via message passing.

Long live long-lived processes!

[GenServers]: https://hexdocs.pm/elixir/GenServer.html
[Supervisors]: https://hexdocs.pm/elixir/Supervisor.html
