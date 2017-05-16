---
layout: post
title:  "Elixir Supervisors and their Strategies"
categories: blog
tags: Elixir, OTP
---

Let's start by briefly answering the question,

## What are Elixir Supervisors?

An elixir supervisor is a process whose sole purpose is to watch other processes
linked to it (children), watch for incoming messages that a process has failed,
trap exits, and respond appropriately. For example, when a child process
crashes, the supervisor can restart the child process, or it can kill and
restart entire sections of our application. If none of that makes sense, take a
quick look at the [introduction to concurrency][intro-to-concurrency] blog post
I wrote a little while ago.

Using supervisors to manage entire sections of our application is an extremely
powerful way to handle errors in Elixir. They allow us to determine how much (or
how little) we want errors to propagate across processes. Indeed, supervisors
are the foundation for Elixir's fault tolerance.

Though we could create supervisor processes from scratch, Elixir (and Erlang)
give us everything we need in OTP! OTP will do the following for us,

* Start and run the supervisor process.
* The supervisor process will traps exits.
* Child processes are started and linked to the supervisor process.
* When a crash happens, the supervisor process receives an exit signal and
   performs corrective actions, such as restarting the crashed process.
* If a supervisor is terminated, child processes are terminated immediately.

And best of all, OTP makes it so easy for us! Let's take a look at how to define the supervisors that come with OTP.

## Supervisor Spec

There are two ways to define supervisors in elixir. The first one is the
_dynamic_ way by using `import Supervisor.Spec` in a function.

```elixir
import Supervisor.Spec

children = [
  worker(WorkerModule, [[:start_link_arg]])
]

{:ok, pid} = Supervisor.start_link(children, strategy: :one_for_one)
```

By importing `Supervisor.Spec` we get a set of convenience functions for
defining children, such as [`worker/3`][worker/3] and
[`supervisor/3`][supervisor/3], for definining the children processes.  We can
then start the supervisor by calling [`Supervisor.start_link/2`][start_link/2]
and passing the children definition.

## Supervisor Behaviour

The second way to create a supervisor is the module-based supervisor created via
`use Supervisor`.

```elixir
defmodule SampleSup do
  use Supervisor

  def init(_) do
    children = [
      worker(WorkerModule, [[:start_link_arg]])
    ]

    supervise(children, strategy: :one_for_one)
  end
end
```

By using `use Supervisor`, we are able to perform additional
actions during initialization in the `init/2` callback. For example, we
may want to set up an ETS table where we could persist data
related to the supervisor's children.

This also allows you to perform partial hot-code swapping of the supervision tree.
In other words, in module-based supervision we could add and remove children directly,
whereas dynamic supervision would require the whole tree to be restarted in
order to perform swaps. Whichever method you use will depend on your needs.

The rest of the module-based supervisor is much like it's dynamic counterpart,
where we define the children with convenience functions and we pass those
children definitions to a function. In the module-based supervisor, we use the
[`supervise/2`][supervise/2] function instead of directly using
`Supervisor.start_link/2`.

## Supervision Strategies

OTP makes it easy to define supervisor processes along with its children. But a
lot of the flexibility of supervision trees in Elixir come from the supervision
strategies. Let's create a sample application with a supervisor so we can take a
look at each supervision strategy in turn.

We will start a
fake Bank application for our demo. If we run `mix new` with the `sup` flag,
`mix` will create a project that gets automatically started with a supervisor,
so let's do that. In the shell,

```shell
mix new bank --sup
```


If you go to bank.ex, you should now see something like this (but with more comments),

```elixir
defmodule Bank do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = []

    opts = [strategy: :one_for_one, name: Bank.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

When `mix` starts our Bank application, it will call that [`start/2`][start/2] function
since that module is using the `Application` behavior.

As you can see, mix is using the `import Supervisor.Spec` because we'll just
start a supervisor when the `mix` starts the application. The supervisor named
`Bank.Supervisor` is the top level supervisor of our application.

Let's define three children,

```elixir
defmodule Bank do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      supervisor(Bank.Account.Supervisor, []),
      supervisor(Bank.Transfer.Supervisor, []),
      worker(Bank.Loan.Processor, [])
    ]

    opts = [strategy: :one_for_one, name: Bank.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

Note that a supervisor can supervise both workers _and_ supervisors.
To do this, we use two convenience functions defined in
`Supervisor.Spec`, [`supervisor/3`][supervisor/3] and [`worker/3`][worker/3].
Since supervisors can supervise other supervisors, OTP allows us to build complex supervision trees where each part of our application is monitored by a process that can handle failures.

Finally, we set the options with a `:one_for_one` strategy and call the
[`Supervisor.start_link/2`][start_link/2] function passing the children and the options.

Let's define some basic modules to stand in for our account supervisor, our transfer
supervisor and for the loan processor.

```elixir
defmodule Bank.Account.Supervisor do
  use Supervisor

  @name :bank_account_supervisor

  def start_link do
    Supervisor.start_link(__MODULE__, [], name: @name)
  end

  def start_child do
    Supervisor.start_child(@name, [])
  end

  def init(_) do
    children = [
      worker(Bank.Account, [])
    ]

    IO.puts ">>> Starting bank account supervisor"
    supervise(children, strategy: :one_for_one)
  end
end
```

```elixir
defmodule Bank.Transfer.Supervisor do
  use Supervisor

  @name :bank_transfer_supervisor

  def start_link do
    Supervisor.start_link(__MODULE__, [], name: @name)
  end

  def init(_) do
    children = [
      worker(Bank.Transfer, [])
    ]

    IO.puts ">>> Starting bank transfer supervisor"
    supervise(children, strategy: :one_for_one)
  end
end
```

```elixir
defmodule Bank.Loan.Processor do
  use GenServer

  @name :bank_loan_processor

  def start_link do
    IO.puts ">>> Starting bank loan processor"
    GenServer.start_link(__MODULE__, [], name: @name)
  end
end
```

Each process will put a message in the console when it is started, and we will use that to verify that the supervision strategies are working as expected.

Let's test it out.

Open up `iex -S mix`,

{% highlight bash %}
$ iex -S mix

>>> Starting bank account supervisor
>>> Starting bank transfer supervisor
>>> Starting bank loan processor

iex>
{% endhighlight %}

In the console, we can already see our Application starting, and along with it the Account and Transfer
Supervisors, as well as the Loan Processor.

To get a better view of our supervision tree, let's take a look at what the erlang observer gives us,

```shell
:observer.start
```

Under the applications tab, we can see our Bank application as well as the entire supervision tree.

We have the Bank.Supervisor as the top level supervisor. It supervises the Account supervisor, the Transfer supervisor, and the loan processor.

### :one_for_one

The `:one_for_one` strategy is the most intuitive strategy. If we have a supervisor _S_
supervising several processes, _x_, _y_, and _z_, and _x_ goes down, only _x_ will be restarted.

Let's take a look at a demo of `:one_for_one` supervision strategy.

Now, for the moment we've been waiting for. Let's test our Bank application's fault-tolerance by taking the Transfer supervisor down.

```elixir
iex> pid = Process.whereis(:bank_transfer_supervisor)
iex> Process.exit(pid, :kill)
```

It may not seem like much, but the message "Starting bank transfer supervisor" tells us that the Bank supervisor
has successfully restarted our Transfer supervisor, and that is cool.

### :one_for_all

The `:one_for_all` strategy tells a supervisor to kill all processes being
supervised when any one of them dies. Thus this is a good strategy to use
when a set of processes depend on each other and should not exist without its
siblings.


In our Bank Application, we have decided to group all things related to a Loan
under a supervisor.

Indeed, if any of the processes fails, it would be better to restart the whole
Loan supervision tree, since they are all so interconnected.

So we'll create a loan supervisor and place it under the Bank Supervisor instead
of the loan processor.

[transition (red arrow)]

Nothing else needs to change here.

Now let's, take a look at the Loan.Supervisor,

[transition]

The Loan supervisor looks familiar, but it has a few differences.

First, let's note that we are using the module format, `use Supervisor`, which in
turn means that we need to define the behavior's callback `init`. We do so here
and define the workers.

Another thing that may be confusing is why we need this `start_link` function here,
since this is not part of the `Supervisor` behavior. This `start_link` is here because
supervisors require their children to have that in order to start them.

So in other words, this `start_link` is required because the Bank.Supervisor is a
parent to this process.

Okay, so let's take a look at it in iex,

```shell
iex -S mix
```

We see that once again all processes were started.

Recall the supervision tree,

```shell
:observer.start
```

We have the Top level supervisor, supervising two other supervisors and one worker.

Back in iex,

As we said, having a `:one_for_all` strategy means that when any process dies, all
of its siblings are terminated and restarted alongside this one.

Let's see it in action:

```shell
pid = Process.whereis(:bank_loan_processor)

Process.exit(pid, :kill)
```

As we can see, no only was the `:bank_loan_processor` restarted but so were all the
other processes. Excellent!

```shell
Process.whereis(:bank_loan_processor)
```

### :rest_for_one

This strategy is similar to the `:one_for_all` in that one process dying could
cause several sibling processes to be killed and restarted. The difference is
that only those processes that were started after the dying process are
restartd. So in a sense, this is like a chain reaction that does not affect
processes that are defined before the process dying.

For the :rest_for_one strategy, all we will do is change the strategy of our
Bank.Supervisor.

[transition (red arrow)]

We would do this if Bank Loan depended on Bank Transfer, and Bank Transfer
depended on Bank Account.

So, let’s take a look at them in `iex`

[transition (next slide)]

[transition (play video)]

```shell
iex -S mix
```

Once again, we see that all of our processes are started in order.

Now remember, `:rest_for_one` means that if Transfer.Supervisor terminates due to
an error, everything after that supervisor will be restarted.

```shell
pid = Process.whereis(:bank_transfer_supervisor)
Process.exit(pid, :kill)
```

There we go. We see the two processes restarting (as well as their children)

```shell
pid = Process.whereis(:bank_transfer_supervisor)
```

### :simple_one_for_one

## Restart Strategies

A last thing, I'd like to highlight is the ability to set the `restart strategy`.

* `:permanent` - the child process is always restarted
* `:temporary` - the child process is never restarted (not even when the
supervisor’s strategy is :rest_for_one or :one_for_all)
* `:transient` - the child process is restarted only if it terminates abnormally,
i.e., with an exit reason other than `:normal`, `:shutdown` or `{:shutdown, term}`

Combining these with the supervision strategies, allows us to create custom strategies
for how to handle errors in our concurrent system.

For example, you may want to dynamically create processes that perform parallel tasks,
but you may not care if one of the workers fails, and you would not want to restart it.
All you want to do is collect the results of the processes that succeeded and continue.

This is something I needed done while working on a search feature in an application.
I wanted to request results from several websites, parse the results, and collect them
into a single unified list. The catch was that linking the processes would cause one of
the website failures to crash the entire search. Instead, I wanted to retrieve the results
that I could and ignore the processes that failed. Using a `:simple_one_for_one` strategy
with `:temporary` restart strategy allowed me to do just that.

[start/2]: https://hexdocs.pm/elixir/Application.html#start/2
[supervisor/3]: https://hexdocs.pm/elixir/Supervisor.Spec.html#supervisor/3
[worker/3]: https://hexdocs.pm/elixir/Supervisor.Spec.html#worker/3
[start_link/2]: https://hexdocs.pm/elixir/Supervisor.html#start_link/2
[supervise/2]: https://hexdocs.pm/elixir/Supervisor.Spec.html#supervise/2
[elixir_concurrency]: http://www.germanvelasco.com/blog/2017/02/27/introduction-to-concurrency-in-elixir.html
[intro-to-concurrency]: {% link _posts/2017-02-27-introduction-to-concurrency-in-elixir.md %}
