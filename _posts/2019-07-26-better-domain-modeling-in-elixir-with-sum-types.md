---
layout: post
title: "Better domain modeling in Elixir with sum types"
date: 2019-07-26
categories: blog
tags: elixir, functional programming, types, good code
excerpt: >
  Sum types are a powerful domain modeling technique. Let’s look at how to use
  them to remove invalid states.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/better-domain-modeling-in-elixir-with-sum-types">thoughtbot.com/blog/better-domain-modeling-in-elixir-with-sum-types</a>.
</div>

Too often, I use structs and maps exclusively to model domains in Elixir. You
might do so too. I think the habit comes from modeling domains in
object-oriented languages and from having a one-to-one mapping between structs
and database records. But lately, I have found [sum types] to be a powerful
domain modeling technique that can help rid projects of bugs caused by invalid
states.

Let's look at an example.

[sum types]: https://en.wikipedia.org/wiki/Tagged_union

## The problem

Some errors in our applications are caused by invalid state -- state we thought
impossible in our application but which is nevertheless present when we find a
bug.

Suppose we are modeling a chess game:

```elixir
defmodule Game do
  defstruct [:status, :players, :winning_player]

  @type status :: :not_started | :in_progress | :finished

  @type t :: %__MODULE__{
    status: status(),
    players: {Player.t(), Player.t()} | nil,
    winning_player: Player.t() | nil
  }
end
```

We have a `Game` struct with three fields:

- The `status` field has three valid values: `:not_started`, `:in_progress`, and
  `:finished`.

- We have a tuple representing the two players, along with the possibility of
  `nil` if the game has not started.

- And we have a field for the `winning_player` when the game finishes, which
  will be `nil` otherwise.

Elsewhere, we have a function that determines the message to be displayed at the
top of a player's screen:

```elixir
def status_message(game) do
  case game.status do
    :not_started ->
      "Waiting for players to join..."

    :in_progress ->
      {player1, player2} = game.players
      "Game on: #{player1.username} vs #{player2.username}."

    :finished ->
      "Player #{game.winning_player.username} wins!"
  end
end
```

One day we receive a bug report saying that when two users finished the game,
they got a 500 error. Looking at our error tracking, we see the exception `**
(UndefinedFunctionError) function nil.username/0 is undefined.` Looking at the
game state, we find this:

```elixir
%Game{
  status: :finished,
  players: { %Player{username: "gandalf"}, %Player{username: "aragorn"} },
  winning_player: nil
}
```

"But that's impossible!" we say. Somehow the game is `:finished` without a
`winning_player`.

Another day our error tracker alerts us to an exception: `** (MatchError) no
match of right hand side value: nil`. Then we get a report saying that two
players cannot start a game. Looking at the game state, we find the following:

```elixir
%Game{
  status: :in_progress,
  players: nil,
  winning_player: nil
}
```

Somehow the game is `:in_progress`, but the players are not assigned. So the
code `{player1, player2} = game.players` in the `status_message/1` function is
throwing a match error.

What can be done? Should we pattern match on more fields?

```elixir
def status_message(game) do
  case {game.status, game.players, game.winning_player} do
    {:not_started, _, _} ->
      "Waiting for players to join..."

    {:in_progress, players, _} when not is_nil(players) ->
      {player1, player2} = players
      "Game on: #{player1.username} vs #{player2.username}."

    {:finished, _, winning_player} when not is_nil(winning_player) ->
      "Player #{winning_player.username} wins!"

    _ ->
      ""
  end
end
```

No. There is too much defensive programming, and we have a strange catch-all
clause at the end that returns an empty message. Those seem like code smells.
After all, we _know_ that a finished game  _should_ have a winning player. And
we _know_ that a game in progress _should_ have two players. That is the
business logic of our chess game. The states we're seeing are invalid, so
let's represent them differently.

## Using sum types

### Restructuring `:finished`

Let us first restructure the case when the game is finished. We know a winning
player is only present if the game finishes, so let's enrich the `status` to use
a tagged tuple for the finished case. The tag `:finished` will continue to mark
the finished status, but the second element of the tuple will now hold the
winning player struct:

```elixir
@type status :: :not_started | :in_progress | {:finished, Player.t()}
```

Now let's remove the `winning_player` from the `Game` struct:

```elixir
defmodule Game do
  defstruct [:status, :players]

  @type status :: :not_started | :in_progress | {:finished, Player.t()}

  @type t :: %__MODULE__{
    status: status(),
    players: {Player.t(), Player.t()} | nil
  }
end
```

And we can improve our `status_message/1` function:

```diff
def status_message(game) do
  case game.status do
    :not_started ->
      "Waiting for players to join..."

    :in_progress ->
      {player1, player2} = game.players
      "Game on: #{player1.username} vs #{player2.username}."


-    :finished ->
-      "Player #{game.winning_player.username} wins!"
+    {:finished, winning_player} ->
+      "Player #{winning_player.username} wins!"
  end
end
```

### Restructuring `:in_progress`

Now let's turn `:in_progress` into another tagged tuple, where the second
element holds the two player structs:

```elixir
@type players :: {Player.t(), Player.t()}
@type status :: :not_started | {:in_progress, players()} | {:finished, Player.t()}
```

We can now remove the `players` field from the `Game` struct:

```elixir
defmodule Game do
  defstruct [:status]

  @type players :: {Player.t(), Player.t()}
  @type status :: :not_started | {:in_progress, players()} | {:finished, Player.t()}

  @type t :: %__MODULE__{status: status()}
end
```

Finally, let's improve our `status_message/1` function again:

```diff
def status_message(game) do
  case game.status do
    :not_started ->
      "Waiting for players to join..."


-    :in_progress ->
-      {player1, player2} = game.players
+    {:in_progress, {player1, player2}} ->
      "Game on: #{player1.username} vs #{player2.username}."

    {:finished, winning_player} ->
      "Player #{winning_player.username} wins!"
  end
end
```

By making our `status` field a sum type, we have modeled our domain more
accurately and removed several invalid states that were causing bugs. Take a
look at our final `status_message/1` function:

```elixir
def status_message(game) do
  case game.status do
    :not_started ->
      "Waiting for players to join..."

    {:in_progress, {player1, player2}} ->
      "Game on: #{player1.username} vs #{player2.username}."

    {:finished, winning_player} ->
      "Player #{winning_player.username} wins!"
  end
end
```

## Theoretical explanation

Both [product types] and sum types are called [algebraic data types] -- structs
falling under the umbrella of product types. So the set of all possible values
for a struct is the [cartesian product] of the values of its fields.

In practice that means that a struct could contain any combination of each of
the possible values of each of its fields. So in order to enumerate all
the possible values of our original `Game` struct, we have to look at all
possible combinations of its fields:

```elixir
# not started
%Game{status: :not_started, players: nil, winning_player: nil}
%Game{status: :not_started, players: {player1, player2}, winning_player: nil}
%Game{status: :not_started, players: {player1, player2}, winning_player: player}
%Game{status: :not_started, players: nil, winning_player: player1}

# in progress
%Game{status: :in_progress, players: nil, winning_player: nil}
%Game{status: :in_progress, players: {player1, player2}, winning_player: nil}
%Game{status: :in_progress, players: {player1, player2}, winning_player: player}
%Game{status: :in_progress, players: nil, winning_player: player}

# finished
%Game{status: :finished, players: nil, winning_player: nil}
%Game{status: :finished, players: {player1, player2}, winning_player: nil}
%Game{status: :finished, players: {player1, player2}, winning_player: player}
%Game{status: :finished, players: nil, winning_player: player}
```

Sum types are also algebraic data types. But the set of all possible values of a
sum type is the [disjoint union] of all of its variants. In practice that means
that our final `Game` struct, with its restructured `status`, could only be one
of the following:

```elixir
%Game{status: :not_started}

# OR

%Game{status: {:in_progress, {player1, player2}}

# OR

%Game{status: {:finished, winning_player}}
```

Using a sum type to model our domain reduced the number of potential (and
invalid) states considerably!

[algebraic data types]: https://en.wikipedia.org/wiki/Algebraic_data_type
[product types]: https://en.wikipedia.org/wiki/Product_type
[cartesian product]: https://en.wikipedia.org/wiki/Cartesian_product
[disjoint union]: https://en.wikipedia.org/wiki/Disjoint_union

## Parting thoughts

If you find the idea that domain modeling can remove certain invalid states
interesting, you might enjoy seeing the same concept applied in other languages.
Richard Feldman gave a [great talk about it in Elm]. Scott Wlaschin [showed the
concept in F#]. And Yaron Minsky, who coined the term "make illegal states
unrepresentable", [explained the concept in OCaml].

[great talk about it in Elm]: https://www.youtube.com/watch?v=IcgmSRJHu_8
[showed the concept in F#]: https://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/
[explained the concept in OCaml]: https://blog.janestreet.com/effective-ml-revisited/
