---
layout: post
title: "Is Elixir a scripting language?"
date: 2018-10-15
categories: blog
tags: elixir, functional-programming
excerpt: >
  Elixir is known for being a language made for building distributed applications
  that scale, are massively concurrent, and have self-healing properties. But is
  Elixir good enough for the mundane scripts of this world?
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/is-elixir-a-scripting-language">thoughtbot.com/blog/is-elixir-a-scripting-language</a>.
</div>

Elixir is known for being a language made for building distributed applications
that scale, are massively concurrent, and have self-healing properties. All of
these adjectives paint Elixir in a grandiose light. And for good reasons!

But is Elixir also a language that can be used for the more mundane tasks of
this world like scripting? I think the answer is a definite _yes_.

To see this, let's take a look at all the ways we can write scripts using
Elixir. We'll build using the same example, going from simple to more complex
solutions.

## Defining the script

Our script will simply create a new markdown file with today's date so we can
write our To Do list. The structure of the directories will be `YYYY/MM/DD.md`,
so we will nest each day under a month and each month under a year.

```elixir
# Get today's date
date = Date.utc_today()
year = Integer.to_string(date.year)
month = Integer.to_string(date.month)
day = Integer.to_string(date.day)

# Generate the month's full path with YYYY/MM format
month_path =
  File.cwd!()
  |> Path.join(year)
  |> Path.join(month)

# Create the month and year directories
File.mkdir_p(month_path)

# Generate the filename with today's date
filename = Path.join(month_path, day) <> ".md"

# Check existence so we don't override a file
unless File.exists?(filename) do
  # include sample header
  header = """
  ---
  date: #{Date.to_string(date)}
  ---

  To Do
  =====

  - What do you need to accomplish today?
  """

  # write to the file
  File.open(filename, [:write], fn file ->
    IO.write(file, header)
  end)
end

# Print out confirmation message
final_message = """

> Created #{Path.relative_to_cwd(filename)}
"""

IO.puts(final_message)
```

Excellent! That is the full extent of our script. Let's now see how we can run
it.

## elixir todo.exs

The first and perhaps simplest way to run an elixir script is just to run
`elixir name_of_file.exs` from the shell. So let's do that.

1. Save the above code in a file named `todo.exs`
2. Run `elixir todo.exs`
3. You should see the following message (with the date you're
   running this script instead of the date I ran it):

```bash
$ elixir todo.exs

> Created 2018/9/28.md
```

If we inspect the file that it created, we can confirm that the template is
there,

```md
---
date: 2018-09-28
---

To Do
=====

- What do you need to accomplish today?
```

## bin/todo

A second way to run the script is to make it into an executable. This is perhaps
an extension of the one above, but I include it as a separate step because I
think it could prove useful in some projects.

Follow these steps:

1. Move `todo.exs` under a `bin/` directory (not required but nice)
1. Mark it as executable with [chmod]: `chmod +x bin/todo.exs`
1. Add a [shebang] `#! /usr/bin/env elixir` at the top of your file
1. Rename it to `bin/todo`

And run `bin/todo`!

If you're unfamiliar with using `chmod` and `shebang` to turn a file into an
executable, take a look at the notes in this [let's build a CLI] video.

[shebang]: https://en.wikipedia.org/wiki/Shebang_(Unix)
[chmod]: https://en.wikipedia.org/wiki/Chmod
[let's build a CLI]: https://thoughtbot.com/upcase/videos/lets-build-a-cli

### When would I use these?

Our `todo.exs` and `bin/todo` scripts are examples of standalone scripts. They
are useful when you want to run a task that is self-contained. `bin/todo` has
the benefit that the user of the script need not know that the script is running
Elixir (though it still needs to have Elixir installed). It's just a script like
any other one you may find in your `bin/` folder.

## mix run todo.exs

Now if you're working with Elixir, it is very likely you are already working in
a [mix project]. If that is the case, `mix` allows you to run arbitrary scripts
via `mix run [name of file]`.

To test this, let's go ahead and create a new project called _tasker_ and run
our script there.

Follow these steps:

1. `mix new tasker`,
1. `cd` into the `tasker` directory,
1. make a `scripts/` directory,
1. copy the `todo.exs` file from the first section into `scripts/todo.exs`

Now run `mix run scripts/todo.exs`!

[mix project]: https://hexdocs.pm/mix/Mix.html#module-mix-project

### When would I use mix?

The benefit of running this script via `mix` (as opposed to the previous two
options) is that the script is part of your project, so it has access to all the
code you have defined in the project.

Let's bring that point home by extracting most of the logic to a `TodoBuilder`
module:

```elixir
# lib/todo_builder.ex
defmodule TodoBuilder do
  def run(date) do
    year = Integer.to_string(date.year)
    month = Integer.to_string(date.month)
    day = Integer.to_string(date.day)

    month_path =
      File.cwd!()
      |> Path.join(year)
      |> Path.join(month)

    File.mkdir_p(month_path)

    filename = Path.join(month_path, day) <> ".md"

    unless File.exists?(filename) do
      header = """
      ---
      date: #{Date.to_string(date)}
      ---

      To Do
      =====

      - What do you need to accomplish today?
      """

      File.open(filename, [:write], fn file ->
        IO.write(file, header)
      end)
    end

    {:ok, filename}
  end
end
```

```shell
# scripts/todo.exs

{:ok, filename} =
  Date.utc_today()
  |> TodoBuilder.run()

final_message = """

> Created #{Path.relative_to_cwd(filename)}
"""

IO.puts(final_message)
```

### Mix run in the wild

If you're interested in seeing this "in the wild", Phoenix [seeds data] in their
applications by running `mix run priv/repo/seeds.exs`.

[seeds data]: https://phoenixframework.org/blog/seeding-data

## mix tasker.todo

Having it as a script that can run with our project code is nice. But sometimes
we want to make it a more explicit part of our project. Turning our script into
a [mix task] can do just that. This is especially true if we expect external
parties to use our project since `mix` tasks have the extra benefit of
documentation!

Let's change our script into a `mix` task. In the `tasker` project,

1. Create a `lib/mix/tasks/` directory
1. Create a `todo.ex` task
1. Move the code we have in `scripts/todo.exs` into that file
1. Add `use Mix.Task` at the top of the module
1. Add a `@shortdoc` and `@moduledoc` with descriptions of what the task does

```elixir
# lib/mix/tasks/todo.ex
defmodule Mix.Tasks.Todo do
  use Mix.Task

  @shortdoc "Creates a new todo file with today's date"

  @moduledoc """
  Creates a new todo file with today's date

  ## Example

  mix todo
  """

  def run(_args) do
    {:ok, filename} =
      Date.utc_today()
      |> TodoBuilder.run()

    final_message = """

    > Created #{Path.relative_to_cwd(filename)}
    """

    Mix.shell().info(final_message)
  end
end
```

Now run `mix todo` and voila!

But that's not all. Note the use of `@shortdoc` and `@moduledoc`. The
documentation that we added in those two module attributes makes our script
especially friendly to other users.

If you check `mix help`, you'll see that our task is listed right after `mix
test`, and it uses `@shortdoc` for its documentation.

![image-of-mix-help](https://images.thoughtbot.com/blog-vellum-image-uploads/nCEFyt65RYOKZVgYlsc5_mix-help.png)

Now check out `mix help todo`. It uses your [@moduledoc] for documentation!

![image-of-mix-help-todo](https://images.thoughtbot.com/blog-vellum-image-uploads/Qh2LB4hsSXiY5yA2sRSC_mix-help-todo.png)

[mix task]: https://hexdocs.pm/mix/Mix.html#module-mix-task
[@moduledoc]: https://hexdocs.pm/mix/Mix.Task.html#module-documentation

### Mix task in the wild

If you're interested on how this gets used in the wild, take a look at how
[Phoenix] and [Ecto] use `mix` tasks for commonly performed actions and for
their generators. Things such as `mix phx.new`, `mix phx.server`, and `mix
ecto.migrate` are all `mix` tasks!

[Phoenix]: https://hexdocs.pm/phoenix/phoenix_mix_tasks.html
[Ecto]: https://hexdocs.pm/ecto/Mix.Tasks.Ecto.html

## ./tasker escript

Wow, it's been a long road but we're finally down to the last way we can script
in Elixir (that I know of).

One of the built-in `mix` tasks that comes with `mix` is [mix escript.build]. It
packages your project and dependencies into a binary that is executable. It even
embeds Elixir as part of the script, so it can be used so long as a machine has
Erlang/OTP _without_ requiring Elixir to be present.

Let's get to it. We have to first update our `mix.exs` file to define an entry
point for the `escript.build` task,

```elixir
# mix.exs
def project do
  [
    app: :tasker,
    # other options
    #
    # add this line below
    escript: escript()
  ]
end

# add the entry point
defp escript do
  [main_module: Tasker.TodoCLI]
end
```

Now let's create `Tasker.TodoCLI` and put our code there,

```elixir
# lib/tasker/todo_cli.ex
defmodule Tasker.TodoCLI do
  def main(_args) do
    {:ok, filename} =
      Date.utc_today()
      |> TodoBuilder.run()

    final_message = """

    > Created #{Path.relative_to_cwd(filename)}
    """

    IO.puts(final_message)
  end
end
```

Now we can simply run `mix escript.build` to build the executable. And you
should see that you have a new executable called `tasker` in the main directory.
Run it with `./tasker` and see the magic happen!

[mix escript.build]: https://hexdocs.pm/mix/master/Mix.Tasks.Escript.Build.html

### When would I use escript?

Much like a `mix` task, an `escript` has the ability to use the rest of your
project's codebase. But it has that ability because the project and its
dependencies are compiled and packaged in. That means you can use it _outside_
of your `mix` project. And since Elixir is embedded, all you need is to have
Erlang/OTP installed. So it makes our little script into an executable that is
easily shareable with others!

To test that, feel free to move the `./tasker` script out of your `mix`
project into another directory and run it. You'll see it does its work just
fine!

## Anything else?

We covered a lot of ground, and we saw that Elixir has great tooling for
creating scripts. But I would be remiss if I didn't include a mention of the
[OptionParser] module. It's a little module that helps parse command line
options from something like this,

```shell
--flag true --option arg1
```

into a keyword list like this,

```elixir
[flag: true, option: "arg1"]
```

So if you're writing Elixir scripts, it's sure to come in handy!

[OptionParser]: https://hexdocs.pm/elixir/OptionParser.html
