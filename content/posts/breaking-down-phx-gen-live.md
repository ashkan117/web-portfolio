---
title: "Breaking Down Phx Gen Live"
date: 2024-04-17T16:15:24+01:00
draft: true
---

## Finding the Mix Task
https://github.com/phoenixframework/phoenix/blob/v1.7.12/lib/mix/tasks/phx.gen.live.ex#L170

## Finding the template
https://github.com/phoenixframework/phoenix/blob/v1.7.12/priv/templates/phx.gen.live/index.ex

## EEX revisited
Given a string sometimes it's nice to inject elixir code. That's a great start but what if we want
to interact with that string? This is where EEX comes in. The [EEX documentation](https://hexdocs.pm/eex/1.12.3/EEx.html#eval_file/3) has a short but sweet example that helps get a gist of things.
```elixir
EEx.eval_string("foo <%= bar %>", bar: "baz")
"foo baz"
```

```elixir
  defp apply_action(socket, :edit, %{"id" => id}) do
    socket
    |> assign(:page_title, "Edit <%= schema.human_singular %>")
    |> assign(:<%= schema.singular %>, <%= inspect context.alias %>.get_<%= schema.singular %>!(id))
  end
```
Taking a look at some of the code we can see why this is useful. Intuitively it makes sense.
We want `Edit + variable_name` and eex helps us with that. The inner details of this are not as 
clear for me. For example I see `schema.human_singular` but how does that match with the binding.

After some trial and error in iex it looks like EEX can take in a binding that has a map value.

```elixir
EEx.eval_string("Edit <%= schema.human_singular %>", schema: %{human_singular: "Todo"})
"Edit Todo"
```

### How Can Inspect Help (Use Inspect for more complex data)
The final question is determining how does inspect come into things. As far as you can see in the 
gen.live example, it looks like it's used when you want to combine things. In the example above,
we can see that since we want to combine some binding with `.get_` we use inspect. However, right
after that it looks like we don't need to use inspect to connect to the end of `.get_`? 

This led me to think it might matter what type of value the binding has? `alias` in this case refers to 
the alias name of a module. For example, if our module name is `ProjectWeb.Controller.UserController`, 
then we'd want the alias to be `UserController`. The use case with the mix.gen.live is that we 
want to determine the alias of a module and add that to the file. Essentially we'd want the 
generator task to properly insert aliases for us.

After more trial and error, it seems that when we pass in a map for a binding is when we need inspect.

```elixir
EEx.eval_string("Edit <%= bar %>", bar: "Todo")
"Edit Todo"
```

But with the map, we need the `inspect` or else the value of `"Todo"` is treated as a literal string.
```elixir
EEx.eval_string("Edit <%= schema.human_singular %>", schema: %{human_singular: "Todo"})
"Edit \"Todo\""

EEx.eval_string("Edit <%= inspect schema.human_singular %>", schema: %{human_singular: "Todo"})
"Edit Todo"
```
The binding is now a map, which is an Elixir data structure. In these cases we need to use inspect so that it is converted to an algebra document. What does that mean? Honestly not sure but the best I can guess is it's more code friendly?

https://hexdocs.pm/elixir/1.15.6/Inspect.html
"The Inspect protocol converts an Elixir data structure into an algebra document."
