---
layout: post
title:  "September Meetup: Introduction to OTP and GenServer processes"
date:   2016-09-06 12:03:00 -0700
---

Hello! Welcome to our September meetup.

## Agenda

* Recap from ElixirConf attendees
* Intro to OTP
* Building out our first app
* Breakout: work on the next features

## Introduction to OTP

### What is OTP?

Open Telecom Platform: Framework, set of libraries for concurrent
process management.

Technically encompasses Erlang, a db called Mnesia, too.

#### All processes have a set of lifecycle methods

![Lifecycle methods](http://learnyousomeerlang.com/static/img/common-pattern.png)
(Source: [Learn You Some Erlang](http://learnyousomeerlang.com/what-is-otp))

#### OTP provides some different categories of tools to help you manage these:

![Where some libraries live](http://learnyousomeerlang.com/static/img/abstraction-layers.png)

### OTP building blocks

**Applications** are groups of
**Processes** that are implented with conventions from
**OTP** (which is a behavior)

### Let's start with the basics

We want to begin modeling our game into constituent "processes".

* Gameplay (stores game state)
* Player (with roles...)
  - Moderator
  - Villager
  - Werewolf
  - Seer

### How should these processes interact with each other?

Remember in our last meetup, we modeled a Websocket connection with a
Channel abstraction. Thus, some extra actors are introduced to the
system:

* GameChannel (nee RoomChannel)

### Let's go over what GenServer does for us

[GenServer](http://elixir-lang.org/docs/stable/elixir/GenServer.html) is a behavior module that abstracts common
client-server patterns. Let's see what this means.

### GenServer Client API

#### `start_link/3`

Starts up a new process to be managed by the calling process.

```elixir
{:ok, pid} = GenServer.start_link(Werewolf.Gameplay, {:meaning, 42})
```

#### `call` and `cast`

Call is a synchronous command that returns a value.

```elixir
GenServer.call(pid, {:join, 'Jason Wells', pid)
```

Cast is an asynchronous broadcast command.

```elixir
GenServer.cast(pid, :ping)
```

### GenServer Server API

#### `init/1`

```elixir
# Client:  GenServer.start_link(__MODULE__, game_id, name: ref(game_id))

def init(game_id) do
  {:ok, %Werewolf.Gameplay{id: game_id}}
end
```

#### `handle_call/3`

Anything that is `call`ed needs to implement `handle_call/3`

```elixir
# Client: GenServer.call(pid, {:join, 'Jason Wells', pid)

def handle_call({:join, player_name, pid}, _from, game) do
  updated_players = game.players ++ [player_name]
  game = %{game | players: updated_players}
  {:reply, {:ok, self}, game}
end
```

#### `handle_cast/3`

Similarly, anything that is `cast`ed needs to implement `handle_cast/3`

```elixir
# Client: GenServer.cast(pid, :ping)

def handle_cast(:ping, _from, game) do
  game = %{game | pinged: true}
  {:noreply, game}
end
```

OK, let's take a look at the code!

By the way, here's a helpful [cheat sheet](https://t.co/KM1rYnxd6i).

### Your friend the Observer

```elixir
iex> :observer.start
```

This gives you a basic overview of what's going on in the Erlang VM.

