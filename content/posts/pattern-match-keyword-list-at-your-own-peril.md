---
title: "Pattern Match Keyword List at Your Own Peril"
date: 2024-04-15T11:34:36+01:00
draft: true
---

## Introduction
Pattern matching with Elixir is used constantly. Keyword lists are also a really common data structure. Therefore it's pretty natural to think you can pattern match keyword lists but as the Elixir docs [point out](https://hexdocs.pm/elixir/keywords-and-maps.html#keyword-lists), "In a nutshell, do not pattern match on keyword lists."

## Order and List Size Matters

```elixir
# Common Sense Matching
# a matches with 5
[a: a] = [a: 5]
# [a: 5]

# Size Matters
# The first one is really saying that match a single item list. 
# Since the right hand side is a two item list this fails.
[a: a] = [a: 5, b: 9]
# MatchError


# Order Matters
# If the order of the match does not match it also fails
[b: b, a: a] = [a: 5, b: 9]
# MatchError

# You care about order but not size
# By pattern matching the rest of the list into a variable
# we can properly match in the cases where we know that :a comes first
[{:a, a} | rest] = [a: 5, b: 9]
# [a: 5, b: 9]

# this fails again since the order does not match
[{:a, a} | rest] = [b: 9, a: 5]

```

### Note on Syntax
A subtle thing to notice is that when we wanted to pattern match using the `|` operator, we used the tuple syntax
that the Keyword list would de-structure to. Essentially the following does not work.
```elixir
 [ a: a | rest] = [a: 5, b: 9]
```
In the case above the parsing becomes unclear. Although to the reader it's still pretty clear that we're trying to use a keyword list, it's not so clear to a computer.
The `|` operator expects a left hand and right hand since the left hand side is not a single grouping 
things are actually a little unclear.

## Best of Both Worlds
There are cases where as the caller of a function, you'd probably prefer Keyword List. There a little easier to type since there is no need to wrap anything in curly braces. After a while they do kind of feel right for option related things.

However as the programmer, pattern matching typically makes it a lot easier to code up optional things like Keyword Lists. Therefore, the natural question becomes, how do we get the best of both? I believe a convention of Elixir is to inside the function convert the Keyword List to a map.

```elixir
Enum.into(options, %{})
# or if you have default values you want to use
Enum.into(options, %{minimum: 50, maximum: 1000})
```
