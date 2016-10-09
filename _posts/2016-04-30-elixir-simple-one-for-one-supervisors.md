---
layout: post
title:  "Elixir simple_one_for_one Supervisors"
categories: blog
tags: Elixir, OTP, Supervisors
---

Supervision trees are awesome. For a while I was a bit confused as to how to use the `simple_one_for_one`
supervision strategy. Let's look at it step by step by creating a simple
bank account from which we can deposit and withdraw money.

Create a mix project: `mix new bank`.

![create_mix_project]({{ site.url }}/images/elixir_supervisors/create_mix_project.png)

Go into the project, and let's start by creating our bank account module in `lib/bank/account.ex`. We'll use Elixir's excellent Agent
abstraction.

![bank_account_module]({{ site.url }}/images/elixir_supervisors/bank_account_module.png)

Our `Account` is very simple. We can deposit money, withdraw money, and check the balance. We "open" or create an account
with our `start_link` and get a `pid` back. Let's take it for a quick spin in `iex`.

![iex_bank_account]({{ site.url }}/images/elixir_supervisors/iex_bank_account.png)

Excellent! Now what we would like to do is to have a supervisor that could supervise any number of bank accounts.

Let's create that supervisor in `lib/bank/account/supervisor.ex`. We'll use the `Supervisor` behaviour.

![bank_account_supervisor_module]({{ site.url }}/images/elixir_supervisors/bank_account_supervisor_module.png)

Awesome. We have a `start_link`, with which we start the supervisor. It'll be retrievable by name (should we want to
reference it without its pid), and when we call `start_account` on it, it will create a new `Bank.Account` child process.
The magic for us is in the `init` function. There we define the supervision strategy `:simple_one_for_one` which allows us
to dynamically attach children to our supervisor.

I also used a `:temporary` restart strategy when declaring the worker, but you don't have to use that. You could
set that to `:normal`. The `:temporary` option makes it so that the supervisor will not restart
the worker when it dies. This was just something I was experimenting with.

Alright, let's take it for a spin. Hit `iex -S mix` in the terminal,

![iex_bank_account_supervisor]({{ site.url }}/images/elixir_supervisors/iex_bank_account_supervisor.png)

Ok, so we have our supervisor and its chilren processes working. But I would like the supervisor to be started automatically when the application starts.
Let's do that!

We're gonna set up our `Bank` module to be automatically started when the application starts. Open up `lib/bank.ex` in
your text editor, and type the following,

![bank_module]({{ site.url }}/images/elixir_supervisors/bank_module.png)

Nice, we use the `Application` behaviour and define the `start` callback. Now all we have left to do is to tell mix to
start this up during application start up. Pop open `mix.exs`, and define a `mod` under `application` function,

![mixfile]({{ site.url }}/images/elixir_supervisors/mixfile.png)

Last step: back to `iex -S mix`! Let's jump in and type `:observer.start`.

![observer_supervisor]({{ site.url }}/images/elixir_supervisors/observer_supervisor.png)

Wow! Our supervisor is there. But... where are our children processes? Well, they're dynamic! Back to the console to
create a bunch of accounts!

![iex_create_accounts]({{ site.url }}/images/elixir_supervisors/iex_create_accounts.png)

Now that we've created them, let's take a look back in the observer window.

![observer_supervisor_children]({{ site.url }}/images/elixir_supervisors/observer_supervisor_children.png)

That is awesome! That has to be the coolest thing you've seen in days.

For more resources, check out the [getting started guide for supervisors](http://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html),
take a look at the [Supervisor documentation](http://elixir-lang.org/docs/stable/elixir/Supervisor.html),
or read [Programming Phoenix](https://pragprog.com/book/phoenix/programming-phoenix).
