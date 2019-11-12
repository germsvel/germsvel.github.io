---
layout: post
title: "Supercharge Your Elixir and Phoenix Navigation with vim-projectionist"
categories: blog
tags: vim, elixir, phoenix
excerpt: >
  Toggle between alternate files, edit files by type, and create files with
  templates &mdash; all with `vim-projectionist`. If you've never used it,
  you're in for a treat.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/supercharge-elixir-and-phoenix-navigation-with-vim-projectionist">thoughtbot.com/blog/supercharge-elixir-and-phoenix-navigation-with-vim-projectionist</a>.
</div>

If you came to Elixir from Rails-land, you might miss the navigation that came
with [vim-rails]. If you're not familiar with it, `vim-rails` creates commands
like `:A`, `:AV`, `:AS`, and `:AT` to quickly toggle between a source file and
its test file and commands like `:Econtroller`, `:Emodel`, and `:Eview` to edit
files based on their type.

The good news is that the same person who made `vim-rails` also made
[vim-projectionist](https://github.com/tpope/vim-projectionist) (thanks Tim
Pope). And with it, we can supercharge our navigation in Elixir and Phoenix just
like we had in Rails with `vim-rails`.

[vim-rails]: https://github.com/tpope/vim-rails

## Projecting Back to the Future

The easiest way to use `vim-projectionist` is to set up projections in a
`.projections.json` file at the root of your project. This is a basic file for
Elixir projections:

```json
{
  "lib/*.ex": {
    "alternate": "test/{}_test.exs",
    "type": "source"
  },
  "test/*_test.exs": {
    "alternate": "lib/{}.ex",
    "type": "test"
  }
}
```

With this configuration, projectionist allows us to alternate between test and
source files using `:A`, and it can open that alternate file in a separate pane
with `:AS` or `:AV`, or if you're a tabs person, in a separate tab with `:AT`.
Note that we define the `"alternate"` both ways so that both the source and test
files have alternates.

If you're wondering how it works, projectionist is grabbing any directory and
files matched by `*` &mdash; from a globbing perspective it acts like `**/*`
&mdash; and expanding it with `{}`. So the alternate of `lib/project/sample.ex`
is `test/project/sample_test.exs` (and vice versa).

With that simple configuration, projectionist also defines two `:E` commands
based on the `"type"`:

* `:Esource project/sample` will open `lib/project/sample.ex`, and
* `:Etest project/sample` will open `test/project/sample_test.exs`.

Pretty neat, right? But wait! There's more.

### Templating

Projectionist has another really interesting feature &mdash; defining templates
to use when creating files. Add the following templates to each projection:

```diff
{
  "lib/*.ex": {
    "alternate": "test/{}_test.exs",
    "type": "source",
+   "template": [
+     "defmodule {camelcase|capitalize|dot} do",
+     "end"
+   ]
  },
  "test/*_test.exs": {
    "alternate": "lib/{}.ex",
    "type": "test",
+   "template": [
+     "defmodule {camelcase|capitalize|dot}Test do",
+     "  use ExUnit.Case, async: true",
+     "",
+     "  alias {camelcase|capitalize|dot}",
+     "end"
+   ]
  }
}
```

The `"template"` key takes an array of strings to use as the template. In them,
projectionist allows us to define a series of transformations that will act upon
whatever is captured by `*`. We use `{camelcase|capitalize|dot}`, so if `*`
captures `project/super_random`, projectionist will do the following
transformations:

* camelcase: `project/super_random` -> `project/superRandom`,
* capitalize: `project/superRandom` -> `Project/SuperRandom`,
* dot: `Project/SuperRandom` -> `Project.SuperRandom`

### Example workflow

Let's put it all together in a sample `MiddleEarth` project.

We can create a new file via `:Esource middle_earth/minas_tirith`. It will
create a file `lib/middle_earth/minas_tirith.ex` with this template:

```elixir
defmodule MiddleEarth.MinasTirith do
end
```

We can then create a test file by attempting to navigate to the (non-existing)
alternate file. Typing `:A` will give us something like this:

```
Create alternate file?
1 /dev/middle_earth/test/middle_earth/minas_tirith_test.exs
Type number and <Enter> or click with mouse (empty cancels):
```

Typing `1` and `<Enter>` will create the test file
`test/middle_earth/minas_tirith_test.exs` with this template:

```elixir
defmodule MiddleEarth.MinasTirithTest do
  use ExUnit.Case, async: true

  alias MiddleEarth.MinasTirith
end
```

Here it is in gif form:

![gif of the flow we just talked about](https://images.thoughtbot.com/blog-vellum-image-uploads/PiQdloI0RG6czTmpWdqB_simple-flow-example.gif)

Very cool, right? But wait. There's more.

## Supercharge Phoenix Navigation

That simple configuration works for Elixir projects. And since Phoenix projects
(beginning with Phoenix 1.3) have their files under `lib/`, it also works okay
for Phoenix projects.

But without further changes, creating a Phoenix controller or a Phoenix channel
will gives us an extra `Controllers` or `Channels` namespace in our modules
because of the directory structure. For example, creating
`lib/project_web/controllers/user_controller.ex` will create a module
`ProjectWeb.Controllers.UserController` instead of the desired
`ProjectWeb.UserController`.

It would also be nice to have controller-specific templates that include `use
ProjectWeb, :controller` in controllers and `use ProjectWeb.ConnCase` in
controller tests (since we always need those `use` declarations). And, it would
be extra nice to have access to an `:Econtroller` command.

We can make that happen by adding Phoenix-specific projections to our
`.projections.json` file. Start with controllers:

```json
{
  "lib/**/controllers/*_controller.ex": {
    "type": "controller",
    "alternate": "test/{dirname}/controllers/{basename}_controller_test.exs",
    "template": [
      "defmodule {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}Controller do",
      "  use {dirname|camelcase|capitalize}, :controller",
      "end"
    ]
  },
  "test/**/controllers/*_controller_test.exs": {
    "alternate": "lib/{dirname}/controllers/{basename}_controller.ex",
    "type": "test",
    "template": [
      "defmodule {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}ControllerTest do",
      "  use {dirname|camelcase|capitalize}.ConnCase, async: true",
      "end"
    ]
  },
  # ... other projections
}
```

Note that these projections no longer use the single `*` matcher for globbing.
They use `**` and `*` separately. And instead of simply using `{}` in alternate
files, they explicitly use `{dirname}` and `{basename}`.

Why the change? Here's what the projectionist documentation says:

> For advanced cases, you can include both globs explicitly:
`"test/**/test_*.rb"`. When expanding with `{}`, the `**` and `*` portions are
joined with a slash.  If necessary, the `dirname` and `basename` expansions can
be used to split the value back apart.

### Controller templates

By separating the globbing, we are able to create templates that do not include
the extra `Controllers` namespace even though the path includes `/controllers`.

We get the project name with `**`, and we get the file name after `/controllers`
with `*_controller.ex`. We then generate the namespace `ProjectWeb` by grabbing
`dirname` (i.e. `project_web`) and putting it through a series of
transformations. Similarly, we generate the rest of the module's name by using
`basename`, putting it through a series of transformations, and appending either
`Controller` or `ControllerTest`.

We are also able to create more helpful controller templates since the
projections are specific to controllers. Note the inclusion of `"  use
{dirname|camelcase|capitalize}, :controller"` and `"  use
{dirname|camelcase|capitalize}.ConnCase, async: true"` in our templates. Our
controllers will now automatically include `use ProjectWeb, :controller` and our
controller tests will automatically include `use ProjectWeb.ConnCase, async:
true`.

### :Econtroller command

Finally, we set the `"type": "controller"`. That gives us the `:Econtroller`
command. We can now create a controller with `:Econtroller project_web/user`.
And for existing controllers, projectionist has smart tab completion. So typing
`:Econtroller user` and hitting tab should expand to `:Econtroller
project_web/user` or give us more options if there are multiple matches.

For example, in the `MiddleEarth` project we can edit the default
`PageController` that ships with Phoenix by using `:Econtroller page` along with
tab completion. And we can create a new `MinasMorgul` controller and controller
test with our fantastic templates by typing `:Econtroller
middle_earth_web/minas_morgul` and then going to its alternate file.

![gif of using :Econtroller to open page controller](https://images.thoughtbot.com/blog-vellum-image-uploads/BXAQIaIGSEaxZ6pgJbFp_e-controller-demo.gif)

## Projecting All the Things

I think you get the gist of it, so I will not go through all the projections.
But just like we added the projections for the controllers, we can do the same
for views, channels, and even feature tests if you frequently write those.

Below I included a sample file to get you started with controllers, views,
channels, and feature tests. Take a look at it. If you prefer it in github-gist
form, [here's a link to
one](https://gist.github.com/germsvel/84caa5395336bffa5911546456fb5d53). The
best thing is that if my sample file does not fit your needs, you can always
adjust it!

If you find any improvements, [I would love to hear about
them](https://twitter.com/germsvel). I'm always looking for better ways to
navigate files.

```json
{
  "lib/**/views/*_view.ex": {
    "type": "view",
    "alternate": "test/{dirname}/views/{basename}_view_test.exs",
    "template": [
      "defmodule {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}View do",
      "  use {dirname|camelcase|capitalize}, :view",
      "end"
    ]
  },
  "test/**/views/*_view_test.exs": {
    "alternate": "lib/{dirname}/views/{basename}_view.ex",
    "type": "test",
    "template": [
      "defmodule {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}ViewTest do",
      "  use ExUnit.Case, async: true",
      "",
      "  alias {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}View",
      "end"
    ]
  },
  "lib/**/controllers/*_controller.ex": {
    "type": "controller",
    "alternate": "test/{dirname}/controllers/{basename}_controller_test.exs",
    "template": [
      "defmodule {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}Controller do",
      "  use {dirname|camelcase|capitalize}, :controller",
      "end"
    ]
  },
  "test/**/controllers/*_controller_test.exs": {
    "alternate": "lib/{dirname}/controllers/{basename}_controller.ex",
    "type": "test",
    "template": [
      "defmodule {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}ControllerTest do",
      "  use {dirname|camelcase|capitalize}.ConnCase, async: true",
      "end"
    ]
  },
  "lib/**/channels/*_channel.ex": {
    "type": "channel",
    "alternate": "test/{dirname}/channels/{basename}_channel_test.exs",
    "template": [
      "defmodule {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}Channel do",
      "  use {dirname|camelcase|capitalize}, :channel",
      "end"
    ]
  },
  "test/**/channels/*_channel_test.exs": {
    "alternate": "lib/{dirname}/channels/{basename}_channel.ex",
    "type": "test",
    "template": [
      "defmodule {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}ChannelTest do",
      "  use {dirname|camelcase|capitalize}.ChannelCase, async: true",
      "",
      "  alias {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}Channel",
      "end"
    ]
  },
  "test/**/features/*_test.exs": {
    "type": "feature",
    "template": [
      "defmodule {dirname|camelcase|capitalize}.{basename|camelcase|capitalize}Test do",
      "  use {dirname|camelcase|capitalize}.FeatureCase, async: true",
      "end"
    ]
  },
  "lib/*.ex": {
    "alternate": "test/{}_test.exs",
    "type": "source",
    "template": [
      "defmodule {camelcase|capitalize|dot} do",
      "end"
    ]
  },
  "test/*_test.exs": {
    "alternate": "lib/{}.ex",
    "type": "test",
    "template": [
      "defmodule {camelcase|capitalize|dot}Test do",
      "  use ExUnit.Case, async: true",
      "",
      "  alias {camelcase|capitalize|dot}",
      "end"
    ]
  }
}
```
