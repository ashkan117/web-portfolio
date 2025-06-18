---
title: "Ecto Stream"
date: 2025-05-17T17:38:05+01:00
draft: true
---

## Ecto Repo.stream

One of the tools that Ecto and Elixir offers but I haven't been confident in
using is Repo.stream. After seeing a [post](https://elixirforum.com/t/how-exactly-an-ecto-stream-works/70872/3)
popup on elixir forum asking the same thing I thought I'd once
and for all really emphasize when and where to use it.

I'm using mainly [this helpful comment](https://elixirforum.com/t/how-exactly-an-ecto-stream-works/70872/4?u=ashkan117)
to elaborate on their points to help myself and hopefully others.

## Large Result Sets

A good indication that `Repo.stream` could come in handy is when you're dealing
with some SQL query that you expect to get **tons** of data back. The reason
this could be an issue is that when you're loading from a DB you are
loading data from disk into memory.

Again let's make this as visual as possible. My Todo application uses
fly.io to host the application. Since I'm using the free tier I'm limited to
1GB of data. My Todo struct looks like the following:

```elixir
  @type t :: %__MODULE__{
          id: Ecto.UUID.t(),
          name: String.t(),
          quadrant: String.t(),
          duration_in_minutes: integer(),
          planned_count: integer(),
          due_date_at: DateTime.t(),
          archived_at: DateTime.t(),
          completed_at: DateTime.t(),
          inserted_at: DateTime.t(),
          completion_note: String.t() | nil,
          description: String.t() | nil
        }
```

And the following uses ChatGPT to break down some of the stats

#### Todo Struct Memory Usage

| Piece                                         |       Approx. size per struct |
| --------------------------------------------- | ----------------------------: |
| **Map overhead** (keys, buckets, pointers)    |                               |
| • map header & per-pair overhead (\~11 pairs) |                               |
|                                               |                      \~ 200 B |
| **UUID string** (36 chars)                    |                       \~ 64 B |
| **“name” & “quadrant”**                       | \~ 32 B total (avg 16 B each) |
| **Integers** (3 fields)                       |     \~ 48 B total (16 B each) |
| **2× Optional strings**                       |      \~ 200 B (varies wildly) |
| **4× DateTime structs**                       |            \~ 4×200 B = 800 B |
| **—— Sum total**                              | ~~1 344 B ≈ 1.3 KB per todo~~ |

### How 1 million Todos could play out

| Avg. size             |  1 user | 10 users | 50 users | 100 users | 500 users | 1 000 users | 5 000 users | 10 000 users |
| --------------------- | ------: | -------: | -------: | --------: | --------: | ----------: | ----------: | -----------: |
| **1.34 KB (1 344 B)** | 798 915 |   79 891 |   15 978 |     7 989 |     1 597 |         798 |         159 |           79 |
| **1.5 KB (1 536 B)**  | 699 050 |   69 905 |   13 981 |     6 990 |     1 398 |         699 |         139 |           69 |
| **2 KB (2 048 B)**    | 524 288 |   52 428 |   10 485 |     5 242 |     1 048 |         524 |         104 |           52 |

## How does Stream help

### Why load everything if you needed only a handful of rows?

By using an `Repo.stream` you can decide to **batch** load the data instead of
loading everything at once, we can load things in batches. The default batch size
is 500 records.

Let's imagine that we want to export all the todos of 50 or
so users to a csv. Or maybe at the end of each day we'd
want to send an email to our users with a summary of their week/day.

Both of these examples could cause a strain on our servers. Remember
that all while these things are happening, you have users
using the app like normal. They should be able to use everything like normal.
Imagine if our email is sent towards the end of the day. This means
that night users will be notice a slower application.

In these situations there is no actual need to have the todos
loaded in memory. We can process the work incrementally.

### Batching

Let's say we want to mass message our users. There's no need to load all
the users in memory at once. We can just do this work in batches.

```elixir
  @doc """
  Streams all users and sends each a message, one at a time.
  """
  def send_mass_messages_simple do
    Repo.transaction(fn ->
      Repo.stream(
        from(u in User, select: u.id),
        max_rows: 500
      )
      |> Stream.each(&simulate_send/1)
      |> Stream.each(&update_message_status/1)
      |> Stream.run()
    end)
  end
```

### Only keep what you need

> "functions like Stream.run/1, Stream.take/2, Stream.filter/2 and Stream.reject/2
> should limit or avoid having all rows returned by the query in memory on your application."

- If you just only to work with a subset of data, use: Stream.filter/2 and Stream.reject/2
- If you only need a known finite amount of data, use: Stream.take/2

## With Great Power Comes Great Responsibility (Don't accidentally DDOS)

You have to be careful on how you handle things in bulk. One issue is that if
you do things the simple way you could be adding more strain to your DB and you
could accidentally [DDOS](https://www.cloudflare.com/learning/ddos/what-is-a-ddos-attack/)
your application.

Or maybe if you need to make request to an external API, then you
might start getting your request rate limit since you're sending
thousands all at once. Maybe they support some bulk update
functionality that you can leverage but if not then you might
need to add some pause to your work.

```elixir
 # we're passing a single item
  |> Stream.each(&update_message_status/1)
```

This code here could be the culprit. This line of code could mean thousands
of individual updates. Instead you should leverage bulk
updates like update_all/2 or update_all/3.

```elixir
 |> Stream.chunk_every(batch_size)
 # now we the argument being passed is a list of
 # of ids instead of
|> Stream.each(&bulk_update_message_status/1)
`k`
```
