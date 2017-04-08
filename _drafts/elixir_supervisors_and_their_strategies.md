---
layout: post
title:  "Elixir Supervisors and their Strategies"
categories: blog
tags: Elixir, OTP
---

A supervisor is an elixir process whose sole purpose is to watch other processes
linked to it, trap exits, watch for incoming messages that a process has
failed, and respond appropriately. For example, when a linked process crashes, the
supervisor can restart the crashed process, or it can kill and restart entire
sections of our application. This is an extremely powerful way to handle errors
in Elixir. It allows us to determine how much (or how little) we want errors to
propagate across processes.

With that knowledge in hand, we could roll up our own supervisors. All we would
have to do is create processes that start children processes, link to them, trap
exits, and respond to error messages. But it is far better to use OTP!

OTP gives us the following for free,

* It starts and runs the supervisor process.
* The supervisor process traps exits.
* From within the supervisor process, child processes are started and linked to
   the supervisor process.
* If a crash happens, the supervisor process receives an exit signal and
   performs corrective actions, such as restarting the crashed process.
* If a supervisor is terminated, child processes are terminated immediately.

And best of all, OTP makes it so easy for us!

## Supervisor Spec

There are two ways to use define supervisors in elixir, the first one is the
_dynamic_ way by using `import Supervisor.Spec` in any function.

{% highlight elixir %}
import Supervisor.Spec

children = [
  worker(WorkerModule, [[:start_link_arg]])
]

{:ok, pid} = Supervisor.start_link(children, strategy: :one_for_one)
{% endhighlight %}

By importing `Supervisor.Spec` we get a set of convenience functions
for defining children (e.g. [`worker/3`][worker/3]). We can then start the
supervisor by calling [`Supervisor.start_link/2`][start_link/2] and passing the
children definition.

## Supervisor Behaviour

The second way to create a supervisor is the module-based supervisor created via
`use Supervisor`.

{% highlight elixir %}
defmodule SampleSup do
  use Supervisor

  def init(_) do
    children = [
      worker(WorkerModule, [[:start_link_arg]])
    ]

    supervise(children, strategy: :one_for_one)
  end
end
{% endhighlight %}

By using `use Supervisor` (the module-based supervisor), we are able to perform additional
actions during initialization since we use the `init/2` callback. For example, we
may want to set up an ETS table where we could persist data
related to the supervisor's children.

This also allows you to perform partial hot-code swapping of the supervision tree.
In other words, in module-based supervision we could add and remove children directly,
whereas dynamic supervision would require the whole tree to be restarted in
order to perform swaps. Whichever method you use will depend on your needs.

The rest of the module-based supervisor is much like it's dynamic counterpart,
where we define the children with convenience functions, but note that
we use the [`supervise/2`][supervise/2] function instead of directly using
`Supervisor.start_link`.

## Supervision Strategies

Another great benefit of using OTP's supervisors is that we get some very interesting
strategies for managing workers. Let's take a look at them.

### :one_for_one

The `:one_for_one` strategy is the most intuitive strategy. If we have a supervisor _S_
supervising several processes, _x_, _y_, and _z_, and _x_ goes down, only _x_ will be restarted.

Let's take a look at a demo of `:one_for_one` supervision strategy. We will start a
fake Bank application for our demo. If we run `mix new` with the `sup` flag,
`mix` will create a project that gets automatically started with a supervisor,
so let's do that. In the shell,

{% highlight elixir %}
mix new bank --sup
{% endhighlight %}


If you go to bank.ex, you should now see something like this (but with more comments),

{% highlight elixir %}
defmodule Bank do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = []
  end

  opts = [strategy: :one_for_one, name: Bank.Supervisor]
  Supervisor.start_link(children, opts)
end
{% endhighlight %}

When `mix` starts our Bank application, it will call that [`start/1`][start/1] function
since that module is using the `Application` behavior.

As you can see, mix is using the `import Supervisor.Spec` because we'll just
start a supervisor when the `mix` starts the application. The supervisor started named
`Bank.Supervisor` is the top level supervisor of our application.

Let's define three children,

{% highlight elixir %}
defmodule Bank do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      supervisor(Bank.Account.Supervisor, []),
      supervisor(Bank.Transfer.Supervisor, []),
      worker(Bank.Loan.Processor, [])
    ]
  end

  opts = [strategy: :one_for_one, name: Bank.Supervisor]
  Supervisor.start_link(children, opts)
end
{% endhighlight %}

We use two convenience functions defined in
`Supervisor.Spec`, [`supervisor/3`][supervisor/3] and [`worker/3`][worker/3].
Note that a supervisor can supervise both workers _and_ supervisors. In this
way, we can build complex supervision trees where each part of our application
is monitored by a process that can handle failures.

Finally, we set the options with a `:one_for_one` strategy and call the
[`Supervisor.start_link/2`][start_link/2] function passing the children and the options.

Let's define some basic modules to stand in for our account supervisor, our transfer
supervisor and for the loan processor.

{% highlight elixir %}
{% endhighlight %}

Each process will put a message in the console when it is started, so that we can
see when they are starting.

Let's test it out.

Open up `iex -S mix`.

See how in the console we already see our Application starting, and along with it the Account and Transfer
Supervisors, as well as the Loan Processor process.

To get a better view of our supervision tree, let's take a look at what the erlang observer gives us,

{% highlight elixir %}
:observer.start
{% endhighlight %}

Notice that under applications, we can see our Bank application.
And here is our supervision tree.

The top level is the Bank.Supervisor.

It supervises the Account supervisor, the transfer supervisor, and the loan processor.

Now, let's test our Bank application's fault-tolerance: say our transfer supervisor goes down.

{% highlight elixir %}
pid = Process.whereis(:bank_transfer_supervisor)
Process.exit(pid, :kill)
{% endhighlight %}

And now we see that the message that the Transfer Supervisor is starting comes in. Thus the Bank.Supervisor
has successfully restarted our Transfer Supervisor.

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

{% highlight elixir %}
iex -S mix
{% endhighlight %}

We see that once again all processes were started.

Recall the supervision tree,

{% highlight elixir %}
:observer.start
{% endhighlight %}

We have the Top level supervisor, supervising two other supervisors and one worker.

Back in iex,

As we said, having a `:one_for_all` strategy means that when any process dies, all
of its siblings are terminated and restarted alongside this one.

Let's see it in action:

{% highlight elixir %}
pid = Process.whereis(:bank_loan_processor)

Process.exit(pid, :kill)
{% endhighlight %}

As we can see, no only was the `:bank_loan_processor` restarted but so were all the
other processes. Excellent!

{% highlight elixir %}
Process.whereis(:bank_loan_processor)
{% endhighlight %}

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

{% highlight elixir %}
iex -S mix
{% endhighlight %}

Once again, we see that all of our processes are started in order.

Now remember, `:rest_for_one` means that if Transfer.Supervisor terminates due to
an error, everything after that supervisor will be restarted.

{% highlight elixir %}
pid = Process.whereis(:bank_transfer_supervisor)
Process.exit(pid, :kill)
{% endhighlight %}

There we go. We see the two processes restarting (as well as their children)

{% highlight elixir %}
pid = Process.whereis(:bank_transfer_supervisor)
{% endhighlight %}

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


[start/1]: google.com
[supervisor/3]: https://hexdocs.pm/elixir/Supervisor.Spec.html#supervisor/3
[worker/3]: https://hexdocs.pm/elixir/Supervisor.Spec.html#worker/3
[start_link/2]: google.com
[supervise/2]: google.com
