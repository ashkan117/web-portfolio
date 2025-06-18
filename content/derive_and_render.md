---
title: "Derive_and_render"
date: 2025-06-10T09:06:39+01:00
draft: true
---

## Derive, JSON, and Jason

Once of the aspects that I didn't/don't truly grok in the Elixir Phoenix world
is when we need to use `@derive` with Jason. Many times I would see the error
think I vaguely understand it, throw in a `@derive` and call it a day.

```elixir
  @derive {Jason.Encoder, only: [:user_id, :organization_id]}
```

Yesterday was not that day. I wanted to quickly prototype a JSON API for a
project I was working on and wanted to return the following Phoenix view.

```elixir
  def render("show.json", %{user: user}) do
    %{
      user_id: user.id,
      message_id: user.message_id,
    }
  end
```

Let's say I wanted to send some aspects of the message associated with the user,
let's assume we've loaded the message already so we don't need to worry
about any `Ecto.Association.NotLoadedError` errors. **I added
the `Message` struct in order to get some autocomplete in my
IDE, what could possibly go wrong?**

```elixir
  def render("show.json", %{user: user}) do
    %{
      user_id: user.id,
      message_id: user.message_id,
      # assume we know that this association is loaded
      message: %Message{
        id: user.message.id,
        body: user.message.body
      }
    }
  end
```

## Error? I have all the data though

At this point I got the you need `@derive` and `Jason.Encoder` error.
I didn't get why though, I preloaded all my data.

This was a new Schema Module so there was no `@derive` defined in the
module. I thought that I didn't need it since in this new Phoenix view I'll just
show only the data I need.

The issue is that Phoenix sees that you're returning a struct, **not a map**.
It will then go to that module to get the instructions on how
do I convert this struct to a map and notices that it doesn't
have that information since we don't have the `@derive` in the module.

The fix is simple enough (in my case I didn't want the `@derive`),
remove the `Message` struct and just return the map.

```elixir
  def render("show.json", %{user: user}) do
    %{
      user_id: user.id,
      message_id: user.message_id,
      # assume we know that this association is loaded
      message: %{
        id: user.message.id,
        body: user.message.body
      }
    }
  end
```

## Conclusion
