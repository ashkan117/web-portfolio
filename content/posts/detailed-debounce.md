---
title: "Detail on Debounce"
date: 2024-10-09T20:44:09+01:00
draft: true
---

## Use Case

The most common use case for needing debounce is there is a series of "events"
that come in rapid succession but it doesn't make sense to get a result for
each event.

Some common examples in the web world for this are

- A user is typing into a search bar, we would want to ideally
  wait for the user to be done typing before doing any work.

## Real World Analogy

### Elevator

I think in general it's easy to view debounce as delaying some work
or grouping multiple things into one. However there are a
surprising amount of ways to delay work (throttle).
Therefore let's really dig into how it delays work. The every day thing
that acts like a debounce is an **elevator with a motion sensor**. Imagine your the
first to walk into an elevator. The elevator will wait a few seconds
(let's say 3 in our case) of no motion
in order to finally close the door. It gives you some grace period.
However let's say you enter then soon after a second person enters.
In this case the elevator will "reset" and wait the full 3 seconds
before closing. Anyone entering the elevator within those 3 seconds will
always reset the elevator timer to its full duration of 3 seconds.

### Public Transportation

Hopefully it's a little more clear that there's a reset mechanism with debounce.
Another example to really drive the point home about its nature is by taking
a look at its "potential" downside. Imagine you're waiting for the bus to work
and you're running late. You're estatic as you see the bus coming earlier
than the timetable says. Who knows maybe if you're lucky you can actually be
on time.

You get on the empty bus and you grab a seat. You sit comfortably in your
seat as you take a sigh of relief. As the door is closing a handful of
people rush in as the bus driver lets them in. You think nothing of it
until a second group of people also rush in causing the doors to reopen.
You think surely this must be the last set of people but midway through that
thought a couple more people come in.

Hopefully you can see where I'm going with this. The point I want to "drive" home
is that there is an potential that depending on the frequency of events, you could
reach a weird bottle neck. In the bus example, you can imagine that if people consistently
enter the bus causing the door mechanism to reset then we have a new bottleneck.
In this case the door will only finally closed after the bus is crammed full of
so many people that there's no way more people can enter. In an input this could
mean that the bottleneck would be that if the user types fast enough then
they could infinitely type with out triggering the debounce event, unless
there are some max number of characters.

> I'm not saying that these edge cases are something that are likely or that
> you should be concerned about them. I'm merely pointing them out since I think
> they help understand more intuitive what debounce does
> more than "it delays an event".

## Code Example

Finally let's take a look at a simple code example using the popular JS utility
library [Lodash](https://lodash.com/) and its
[debounce](https://lodash.com/docs/4.17.15#debounce) function.
**We use the debounce function to create a debounced function of what we
actually want to do.** It's a wrapper. You give it the work that you
ultimately want to be done, but you're delegating the logic of determining
when to "smartly" call the function to the lodash function.

```ts
const onSearch = (input: string) => {workHere()}
// 1000 is the grace window, any event that enters
// during will reset the window
const debouncedOnSearch = debounce(onSearch, 1000))
// note that the debounced function takes the same function
// signature of the original function
<Input onType={(input: string) =>  debouncedOnSearch(input)} />
```

It's a fairly nice abstraction, you treat things as though you're calling your
original function but you get the nice guarantee that it will group all the
calls together in a reasonable way.

> The code above is pseudo-code. In React this will not work due to the way
> things rerender. Take a look at my follow article
> [Why isn't my debounce working in my React App](/posts/debounce-usecallback)

## Naming of Things

A funny reason why debounce was harder to understand in the beginiing was
funny enough because of the naming of things. I think for most people, we learn
about debounce from some random post from stackoverflow post. It goes something along
the lines of them saying use function called debounce with a library with a weird
name called lodash. What is bouncing, what the heck does lodash mean?

Below are some links for the curious.

- [Where Loadash Gets its Name](https://stackoverflow.com/questions/38006384/why-the-name-underscore-or-lodash)
- [Throttling vs Debounce](https://css-tricks.com/debouncing-throttling-explained-examples)
- [Why is Called Throttle and Debounce](https://codepen.io/tigt/post/why-are-they-called-throttling-and-debouncing-anyway)
