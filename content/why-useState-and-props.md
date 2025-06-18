---
title: "Why UseState and Props"
date: 2025-06-18T09:05:25+01:00
draft: true
---

## Background

I have run into this problem a few times and I think finally the pattern
clicked on why it happens.

I have talked about this before in [React Initializing Usestate With Props](https://ash.ms/posts/react-initializing-usestate-with-props)
which itself just builds on top of [Putting Props to UseState](https://tkdodo.eu/blog/putting-props-to-use-state).

This blog just focuses on the interplay between the parent
and child component that causes this scenario to happen.

## The Child naturally has state

The child component naturally has some state that it needs to manage.
This is the case for all complex components. At some point a "dumb"
component continuously grows in complexity and eventually it needs to manage
its own state.

## The Parent has some conditional logic

The problem occurs when the parent component working with the child
component has some conditional logic that changes the state of things.

### Detour to explain the problem

Let me take you on a journey to the most recent instance I ran into this.
We have a complex dropdown that manages its own state. It manages at the
very least

- What items are selected
- How the labels are displayed

Let's imagine that we're showing a list of projects to choose from.
The parent wants to use this complex dropdown. The parent wants the
ability to filter the options shown in this dropdown. If the user
selects a tag, then we only want to show the projects with that tag.

Therefore,

- some props passed to the child component are determined by the parent component.
- the child component is complex enough where it must manage its own state.

Finally the issue has highlighted in the previous articles is that the
`useState` hook is initialized once. So although the props change,
since the component is not un-mounted, the initial state
is locked in.

## Is this an anti-pattern?

I don't think it is. I think this case where the parent wants to control some things
but the child component must manage its own state is a perfectly valid use case.
However, I think the main thing is to really consider if this is a requirement
of the components. Mainly, **is the child component really complex enough to
warrant managing its own state?**

In my case, the dropdown was probably simple enough not to warrant
managing its own selected items state.
