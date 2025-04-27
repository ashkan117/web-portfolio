---
title: "React Tldr"
date: 2025-04-24T11:34:59+01:00
draft: true
---

## Mandatory Reading

- <https://www.developerway.com/posts/react-re-renders-guide#part2.5>
- <https://www.joshwcomeau.com/react/why-react-re-renders/>

## When a component changes why does react re-render it's children/desendants?

Take a look at the visual showing this in [Josh Comeau's post](https://www.developerway.com/posts/react-re-renders-guide#part2.5).
So now hopefully this stays in your head as a fact but I just want to draw out
a couple more observations that I always need to remind myself of

1. Re-render doesn't mean anything will visually change
2. This re-rendering of the children/desendants is due to how state flows in React

The first point is just easy to forget since re-render is an overloaded term.

The second is more important in this case. In React the flow of data is called
"one-way data flow because the data flows down from the top-level
component to the ones at the bottom of the tree." as stated in
the [docs](https://react.dev/learn/thinking-in-react#step-4-identify-where-your-state-should-live)

So if somewhere in the top changes, it's a reasonable assumption that
the children **could** need to re-render since it's **dependant**
on the parent. This is the default behaviour but then you can optimize
**if needed or when necessary** with memoization.

## My useState that depends on a prop doesn't change on props changes

You must remember that the `initialState` is "locked in" when
the component **mounts** for the first time. On every re-render
it will ignore the value.

### Solution: Move the State Up?

### Solution
