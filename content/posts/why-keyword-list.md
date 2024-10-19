---
title: "Why Keyword List"
date: 2024-05-01T12:01:22+01:00
draft: false
---

## Intro
As an Elixir beginner Keyword Lists are one of the weird features of the language. Coming from other languages, you'd almost always think of map-like structure first. So there's a gnawing feeling, why keyword list? We'll start with the unique characteristics of keyword list then try to go into practical use cases that leverage the unique characteristics which are:

- Order
- Duplicates

## Order
Keyword List preserve user ordering which could be useful. There are a few practical applications of this that could be useful when you're developing a project. One application is simpler, in certain situations order does matter and must be enfored. As [Jose Valim mentions](https://elixirforum.com/t/why-keyword-lists-for-opts-instead-of-maps/29654/12), Elixir uses this ordering to help warn when the `try/catch/after` block is in the wrong order. 

Another similar application of keyword lists are when ordering of the lists has special meanings. [first, second] might mean something different than [second, first ]. I haven't run into a strong use case for this so once I do I'll return back to this.

### WARNING: Keyword methods do not promise order
A really important disclaimer is that although the list itself preserves order, the functions in [Keyword](https://hexdocs.pm/elixir/Keyword.html#content) do not as stated in the [docs](https://hexdocs.pm/elixir/Keyword.html#module-duplicate-keys-and-ordering). 

## Duplicates Allowed

In the same Elixir Forum comment, Jose Valim points out that since Keyword lists allow for duplicates, this allows us to do something like `import Keyword, only: [get: 2, get: 3]`. In this case we're able to specify the function name we're interested in and since we can "overload" functions in Elixir we can use the **arity** to distinguish between which function we mean.

Another example of this that I ran into was working with repeated query values in an HTTP GET request. Sometimes when working with APIs you'll notice a request that works with a list of data. There's a lot of ways this shows up. Sometimes, since the data you're requesting is specified in a complex way people use a POST request to retrieve data. This makes the request simpler but then you're using POST for a reason it wasn't really meant. Another is to keep it a GET request but the downside is then to specify what we want must be in the query params since GET cannot have a request body. Ignoring which way is better, let's focus on the get request that looks like the following.
```bash
/recipes?ingredient[]=chicken&ingredient[]=paprika
```

My goal was given a list of ingredients (`['chicken', 'paprika']`) , how can I create this request? Well in this case Map could not work because the key is the same, ingredient. **This highlights a case where Map just plain would not work.** So Keyword Lists comes to save us since we place the same key multiple times in a Keyword Lists. Let's finish out this example. 

So I wanted my key to be `ingredient[]`. We could have kept it simpler but in this case it makes for a good exercise since we'll briefly learn how to make a complex atom key.

The remaining part is specifying complex atoms. Disclaimer about ETS here.
I wanted the key to be ingredient[] which is not a valid atom?
```elixir
"ingredient[]": "chicken"
```


## Bonus: Pushback
- Try ecto with maps instead of KW… You won’t have much joy…
- Historical 

## References
- https://elixirforum.com/t/why-keyword-lists-for-opts-instead-of-maps/29654/11
