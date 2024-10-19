---
title: "Uncontrolled/Controlled Components I: A Mixed Story"
date: 2024-10-19T11:23:24+01:00
draft: false
---

## Note to the Reader

This post is not really meant for people discovering this concept for the first time.
It's really meant for people who have heard about it already but it still hasn't
clicked.
Maybe they've tried to implement this concept themselves but it didn't feel natural.
Hopefully after that initial experience can they come back and find this
post and it clicks.

## Introduction

[The React documentation](https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components)
does a good job explaining it but I think to someone who is first learning this
some of the nuance is easy to miss. Let's see what they say and I'll highlight
some things that stood out to me.

We'll go through an [Accordion and Panel](https://codesandbox.io/p/sandbox/9r8r22)
example that React uses and expand on things that I think are worthwhile.

## Local State vs Props

The first thing that React really mentions is that **props** and **local state**
have an important role.
But they later go to clarify that there is nuance there.

> "... [Y]ou might say a component is “controlled” when the important information
> in it is **driven** by **props** rather than its **own local state**.
> This lets the parent component fully specify its behavior."

### Who is Driving?

It's absolutely crucial to understand the relationship between your
component and its parent.

- Does the parent care about managing state?
- Does it care about any of the state?

Well if the parent does care about the state, how does it care?

- Does it just want to **listen**?
- Does it want to **affect** the relationship?

### Controlled Component means it has props?

In the first part of the quote, it says that "controlled" components are driven
by their props. So what about the `<Panel>` component inside the `<Accordion>`
component here? Is the Panel controlled or uncontrolled?

```jsx
export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About">
        With a population of about 2 million, Almaty is ...
      </Panel>
      <Panel title="Etymology">The name comes from ...</Panel>
    </>
  );
}
```

Well it looks like it uses props so it's controlled then right?
Not quite. This is uncontrolled since the crucial behaviour of the component,
the opening and closing of the component, is not controllable (through props)
in the parent component.
Let's go over a different section of the React Docs to dig in.

## You choose what's uncontrolled and controlled

The story is a lot more blurred than it might seem then? It's really
about **choosing** what information is important to control. In
the example of the `<Accordion>` and the `<Panel>` above let's look
at the nuance

- The title is controlled by prop through the parent
- The state of isActive/isOpen is uncontrolled
  - Panel controls this through its local state

The docs say the following...

> "In practice, “controlled” and “uncontrolled” aren’t strict technical
> terms—each component usually has some mix of both local state and props.
> However, this is a useful way to talk about how components are designed
> and what capabilities they offer.
> ...\
> When writing a component, consider which information in it should be
> controlled (via props), and which information should be uncontrolled
> (via state)."

## Component Design

This really returns us back to component design. **How do we want this
component to be used in the parent?**

### What a Controlled Panel looks like

In the case of a panel, we might consider the behaviour of opening
and closing the thing that we want to be uncontrolled or controlled.
If we want it to be controlled, then that would mean that the parent
would the one in charge of things.

```jsx
export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        With a population of about 2 million, Almaty ...
      </Panel>
    </>
  );
}
```

## Looking at the Panel Component Internals

Finally let's just look at how the Panel component looks like from the inside.

### Controlled relies on props

```jsx
function Panel({ title, children, isActive, onShow }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? <p>{children}</p> : <button onClick={onShow}>Show</button>}
    </section>
  );
}
```

### Uncontrolled relies on local state

```jsx
function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>Show</button>
      )}
    </section>
  );
}
```

## Closing Remarks

The goal of this post is to highlight that local state and props play
an important role but remember that the story is mixed. **Even though
a controlled component is controlled through props, that doesn't mean
that it won't have local state (useState) in that component.** Likewise,
although an uncontrolled component relies on local state, that doesn't
mean that it won't accept props.

The more advanced the component, the harder it is to determine what
what should be controlled or uncontrolled. A really common thing
that happens is that you create a component and through feature changes
it needs to be controlled. I'll end this post with a few rules of thumb

- controlled components are more flexible
  - They can do whatever you want
- uncontrolled components are simpler
  - They take care of the unimportant things for you
