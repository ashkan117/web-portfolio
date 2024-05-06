---
title: "Why Keyword List"
date: 2024-05-01T12:01:22+01:00
draft: true
---

## Intro
As an Elixir beginner Keyword Lists are one of the weird features of the language. Coming from other languages, you'd almost always think of map-like structure first. So there's a gnawing feeling, why keyword list? We'll start with the unique characteristics of keyword list then try to go into practical use cases that leverage the unique characteristics which are:

- Order
- Duplicates

## Unique Characteristics of Keyword List

### Order

### Enforcing/Preserving Order
Keyword List preserve user ordering which could be useful. As [Jose Valim mentions](https://elixirforum.com/t/why-keyword-lists-for-opts-instead-of-maps/29654/12), Elixir uses this ordering to help warn when the `try/catch/after` block is in the wrong order. This kind of highlights that in some cases you do want to enforce order. 

## Duplicates Allowed

Iin the saem Elixir Forum comment, Jose Valim points out that since Keyword lists allow for duplicates, this allows us to do something like `import Keyword, only: [get: 2, get: 3]`. In this case we're able to specify the function name we're interested in and since we can "overload" functions in Elixir we can use the **arity** to distinguish between which function we mean.

Another example of this that I ran into was working with repeated query values in an HTTP GET request. Sometimes when working with APIs you'll notice a request that works with a list of data. There's a lot of ways this shows up. Sometimes, since the data you're requesting is specified in a complex way people use a POST request to retrieve data. Makes the request simpler but then you're using POST for a reason it wasn't really meant. Another is to keep it a GET request but the downside is then to specify what we want must be in the query params since GET cannot have a request body. Ignoring which way is better, let's focus on the get request that looks like the following.
```bash
/recipes?ingredient[]=chicken&ingredient[]=paprika
```

My goal was given a list of ingredients, how can I create this request? Well in this case Map could not work because the key is the same, ingredient. This highlights a case where Map just plain would not work.

The remaining part is specifying complex atoms. Disclaimer about ETS here.
I wanted the key to be ingredient[] which is not a valid atom?
```elixir
"ingredient[]": "chicken"
```

### WARNING: Keyword methods do not promise order
A really important disclaimer is that although the list itself preserves order, the functions in [Keyword](https://hexdocs.pm/elixir/Keyword.html#content) do not. https://hexdocs.pm/elixir/Keyword.html#module-duplicate-keys-and-ordering

## Bonus: Pushback
- Try ecto with maps instead of KW… You won’t have much joy…
- Historical 

## References
- https://elixirforum.com/t/why-keyword-lists-for-opts-instead-of-maps/29654/11
