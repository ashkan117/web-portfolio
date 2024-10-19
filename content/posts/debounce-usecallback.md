---
title: "Why your Debounce isn't working in React (useCallback)"
date: 2024-10-09T22:32:08+01:00
draft: true
---

## Motivation

Hopefully the [debounce article](/posts/detailed-debounce) helps explain the
ins and outs of debounce.
You're excited to use it in your React app but it's not working. Wait what?
It's not working?! The concept is simple enough so what's the issue!

I swear your almost there. There's one niche thing tied to the way react works that
you need to account for.

## React and Re-Render on Type

```ts
const onSearch = (input: string) => {workHere()}
const debouncedOnSearch = debounce(onSearch, 1000))
<Input onType={(input: string) =>  debouncedOnSearch(input)} />
```

It's very subtle but the issue is that `onSearch` is the function we're
trying to debounce yet every time React re-renders it creates a new
version of your function. This means before you type anything you have V1
of `onSearch`. Let's call that `onSearchV1`. When you type a character this cases
React to re-render which then creates a new version of `onSearch`, `onSearchV2`.
Let's say we type one more character which ends us with `onSearchV2`.

Note that we end up with three debounced versions of these as well but
the issue is that
any `onSearchV1` is lost. There's no way to directly call it. Therefore, even though
it's still a debounced function, there is no way to call that specific version
of it
and after 1000 milliseconds it will trigger the work. The same goes for `onSearchV2`
once you type the third character. There's no way to group up input events to go
towards `onSearchV2` which means after another 1000 milliseconds you get another
call to the work of `onSearch`.

## How to cache a function definition between re-renders?

Reacts solution to this is [useCallback](https://react.dev/reference/react/useCallback).
Luckily in our case we have a simple version of this since the
**function definition** of our `onSearch` should stay the same. When the input
