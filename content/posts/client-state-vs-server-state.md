---
title: "Client State vs Server State"
date: 2025-03-06T09:21:25Z
draft: true
---

## Things I wish I knew

I think fundamentally this is such a key element in any frontend application
regardless on the framework(s)/language(s) you are using but it's one
that is rarely explicitly stated.

- Making data from server available everywhere
- Becareful when connecting your API calls to Global State
- If not global state, then what
- Caching details

## "Fetch data from the server and make it available everywhere."

I have had a ton of back and forth of whether introducing React Query to our
existing React application at work is worth it. I feel like fundamentally
what we've been doing at work is different so introducing a "new way"
of doing things would mean that the codebase would be more fragmented
cause components would continuously look different.
Regardless, this one line from [a maintainer of React Query](https://tkdodo.eu/blog/practical-react-query#client-state-vs-server-state)
really hit home. "Fetch the data from the server and make it available everywhere".

## Reactivity and Optimization

What are the common reasons that would we want to make it available everywhere?
Well to me there are two main ones.

1. Reactivity
2. Optimization

You can imagine that we have a Dashboard application that uses the API request
to show different forms of similar data. Let's use the lovely [Salad UI Demo](https://github.com/bluzky/salad_ui)
as an example.
![SaladUI Demo Application](/images/client-state-vs-server-state/salad-ui-dashboard.jpg)

We can imagine that maybe this weeks and this months components are independent
and using the same API request behind the scenes. Intuitively we can think
that as soon as one component makes the request, we have same information
that the second component wants so we can reuse things there. This
goes to the thinking that I want a handful of components to "react"
to the same data.

Similarly, we can also think that as soon we make one request, we
already have the data so there's no need to make the request again.
Let's save a request in our API.

> I want to make a note that the server in many applications is the
> source of truth. Let's think about the case that I request this
> weeks orders, then another user makes an order. A "downside"
> of not making that request is that we won't see this update to date
> information of our orders. That's why something like a dashboard
> would actually want more of a realtime system rather than the
> typical HTTP request response flow.

## Wrongly updating Global State

It became a pattern in an application that I work on that
we have a React Context that holds our state but also methods
that interact with the requesting data from our servers. These
methods would:

1. Make the API call
2. Update our global state (so that its available everywhere)

There are downsides to this which libraries go to address but I
think the more major concern is not realizing that some API
calls should not update the Global state.

### Filtered Data

Let's imagine we have some global state in a context called
`orders`. Not all API requests should update our global state.
If we always think that API requests should be coupled
to global state, we run into the following problem.

Let's say we click on search our orders and a modal appears
with a search input. This search input should help us filter
through our list of orders,but **it should not update `orders`**.

I think it's reasonable where no other component is really listening
to these results. More importantly this means that if you updated
`orders` this means that when the user goes back to the dashboard to see
all orders, the state now actually shows the filtered orders! That's a bug!

In the project I have been working on this does happen a few times.
Something (it's hard to debug who actually updates the state), triggers
an API call which then triggers global state in an undesired way.

## Who Owns the Data Server or Client?

When you make API requests you do not own the data! Again I'm going to quote
TKDodo here.

> "So it seems that we have always been treating this server state like any other
> client state. Except that when it comes to server state (think: A list of
> articles that you fetch, the details of a User you want to display, ...),
> your app does not own it. We have only borrowed it to display the most
> recent version of it on the screen for the user. It is the server who owns
> the data.

In non real time applications you are always borrowing data. The server holds
the source of truth and as soon as you make a request you for a split second
get the most up to date data. The second you received it, any other
client/user can change the data and you'd never know.

In the view of React Query and similar libraries,
by "saving" server data in your client side state or global
state, you are attempting to own data that will change without your control.

- If a user logs in and goes to their preferences/settings
  (light theme, fonts, birth-date), absolutely no one can change
  that information. They own it. It's safe to save it in
  Global State
- If I go to my user feed on Facebook and Twitter, I don't own
  that data. That's controlled by other people making posts but
  also Facebook's and Twitter's algorithms. This means that even
  if I don't make any post or anyone in the world makes any posts.
  The feed can still reasonably change if the company's recommendation
  algorithm changes.

## Managing Cache vs State

Finally let's enter the solution that React Query and other libraries
settled on.

> To me, that introduced a paradigm shift in how to think about data.
> If we can leverage the cache to display data that we do not own, there
> isn't really much left that is real client state that also needs to
> be made available to the whole app."

Their solution views the data as an ever changing cache. This view does
seem like a more accurate picture of the flow of data but keep in mind that
caching is a notoriously difficult problem. However there are simple/common
strategies that show up.

1. If the client makes tries to update data that we don't own, we know
   that we can invalidate the cache.

## Closing Remarks: Navigating the World of Query libraries (Stale While Revalidate)

In order to better understand this, take a look at

- <https://swr.vercel.app/>
-

"SWR is a strategy to first return the data from cache (stale),
then send the fetch request (revalidate),
and finally come with the up-to-date data."
