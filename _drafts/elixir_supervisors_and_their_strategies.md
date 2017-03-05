---
layout: post
title:  "Elixir Supervisors and their Strategies"
categories: blog
tags: Elixir, OTP
---

A supervisor is an elixir process whose sole purpose is to watch other processes
linked to it, trap exits, and watch for incoming messages that a process has
failed. When a linked process fails, the supervisor can perform a number of things,
such as starting a new process.

This is an extremely powerful tool to handle errors in Elixir. When an error occurs,
it can propagate only as far as we want, and then we can restart parts of the system
to a clean state.

This is the cornerstone of elixir's fault tolerance. Supervisors are able to kill
and restart sections of an application.

## Supervisor Behaviour

We could write our own supervisors by staring "child" processes with 'spawn_link',
then 'trap_exit', and expecting error messages. But it is better to use the behaviour
defined in Elixir. It ensures compatibility with OTP, and they have better error handling.

Like any other behavior in Elixir, we could use `@behaviour Supervisor`. But it is
even better to use `use Supervisor`.

There are two ways to use the Supervisor behavior.

### `import Supervisor.Spec`

{% highlight elixir %}
import Supervisor.Spec

children = [
  worker(WorkerModule, [[:start_link_arg]])
]

{:ok, pid} = Supervisor.start_link(children, strategy: :one_for_one)
{% endhighlight %}

By importing `Supervisor.Spec` we get a set of convenient functions to create the supervisor.

### `use Supervisor`

{% highlight elixir %}
defmodule SampleSup do
  use Supervisor

  def start_link do
    Supervisor.start_link(__MODULE__, [])
  end

  def init(_) do
    children = [
      worker(WorkerModule, [[:start_link_arg]])
    ]

    supervise(children, strategy: :one_for_one)
  end
end
{% endhighlight %}

By using `use Supervisor` (the module-based supervisor), we are able to create a supervisor
with a name (in start_link), as well as perform additional actions during initialization.
For example, we may want to set up an ETS table (redis like table) that would be related to
our supervisors.

This also allows you to perform partial hot-code swapping of the tree; eg. adding or removing
children - module-based supervision will add and remove the children directly.
Dynamic supervision (#1) would require the whole tree to be restarted in order to perform swaps.

Whichever method you use will depend on your needs.

## Supervision Strategies

Another great benefit of using OTP's Supervisor behavior is that we get some very interesting
strategies for managing workers for free. Let's take a look at them.

### :one_for_one

The `:one_for_one` strategy is the easiest to understand. It simply means that
if your supervisor supervises several processes and one goes down, only that one
will be restarted.

Let's take a look at a demo of :one_for_one supervision strategy.

We have a fake Bank application.

`mix` will call start an Application by calling the `start` function in a module using the `Application`
behavior. And that's what we have here.

[image of Bank.ex]

As you can see, we are using the `import Supervisor.Spec` because we'll just start a supervisor when the
`mix` starts the application.

This is our top level supervisor.

We provide three children. And note, that we are using two convenience functions defined in `Supervisor.Spec`,
namely `supervisor/2` and `worker/2`.

Finally, we set the options with a `:one_for_one` strategy and call the `Supervisor.start_link/2`
function passing the children and the options.

Each process will put a message in the console when it is started, so that we can see when they are starting.

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
