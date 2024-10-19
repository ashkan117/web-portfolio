---
title: "Flavors of Controlled and Uncontrolled"
date: 2024-10-19T13:29:28+01:00
draft: true
---

## Conventions of Controlled and Uncontrolled Components

Let's quickly talk about some conventions that you might notice in
frontend component libraries like
[Radix](https://www.radix-ui.com/).

If you need a little more details you can take a look at [My Old Post](/posts/uncontrolled-controlled-components-mixed-story/)
. Also a shout out to [Hari's post](https://dev.to/haribhandari/uncontrolled-components-controlled-components-598o)
which helped things click after seeing the ways that component can be uncontrolled
or controlled. My post aims to highlight when you'd want one flavor over another.

> An implicit assumption is that the components we're working can (depending
> on the props)
> be an uncontrolled or controlled component (not at the same time).
> This pattern is used in
> component libraries like Radix. It's useful in situations where maybe
> from your requirements
> you make a component that is initially uncontrolled but later need some additional
> flexibility so you "extend" it to be controlled.

## All possible combinations

You'll find that there are 3 props that come to play

- value
- defaultValue
- onChange

Having the `value` prop or not determines whether the component is
controlled or uncontrolled!

```html
// uncontrolled
<input />
<input defaultValue="John Doe" /> // uncontrolled
<input defaultValue="John Doe" onChange="{doSomething}" />

// controlled
<input value="John Doe" /> // Bad Practice, React warning
<input value="{value}" onChange="{doSomething}" />
<input value="{value}" defaultValue="John doe" onChange="{doSomething}" />
// Bad Practice, React Warning
```

> These are taken from [Hari's post](https://dev.to/haribhandari/uncontrolled-components-controlled-components-598o)
> but we'll expand on them more below

## Subtle Nuances

### Bad Practice: Why Value Alone is ambiguous

At some point in your React career you might see the following error.

![onchange-or-readonly](/images/onchange-missing-warning.png)

What it's really trying to tell you is that if you have no `onChange` prop
then the value of this component can never change.

In that case would you want this component to be read only? If yes, then
you really should explicitly set the `readOnly` (exists since native HTML has `readonly`).
Otherwise, that means you do want the value to be able to change so in that case
please provide an `onChange` function so we can know how to do that.

### Bad Practice: Default Value + Value leads to ambiguity

Another error you might see in your React career is this one

![uncontrolled-component-react-warning](/images/uncontrolled-component-warning.png)

What it's really trying to tell you is

1. React considers a component with a `value` prop a controlled component
2. We noticed you also passed a `defaultValue`

The final thing you need to notice is that there could be an **ambiguous**
state. What if `defaultValue != value`? In this case which value should we have?
React chooses to take `value` which makes the component controlled, but it
gives you a notice of this ambiguity.

### Why Default Value over Value

I think one thing that seems arbitrary at first but has a surprising amount
of depth is why have `defaultValue` in the following **uncontrolled** case?

```html
<input defaultValue="John Doe" onChange="{...}" />

<input value="John Doe" onChange="{...}" />
```

The [value](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#value)
is a native attribute in html while `defaultValue` is for JS Frontend libraries.
This is because JS Frontend have some concept of state that's different than the
native HTML world. In these libraries, like React, state management is most of
the work. So how does React know if it should be responsible for state
or if the Browser should be? It uses `value`. If `value` is present then
React will take over the responsibility (making it controlled).
Otherwise, we'll let the Browser handle things.

## When to use what

Let's go over the valid cases...

| Input Variant                                              | Reason for Use                                                                                                                                                                                                                                                                                                                                                                                  |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `<input onChange={doSomething} />`                         | **Uncontrolled with side effects**: This allows the input to be uncontrolled, but React can still listen for changes to trigger other effects. Useful if you don't need React to manage the value but still need to handle change events. You can use this for sitations where you might want to send HTTP requests on some input change.                                                       |
| `<input value={value} onChange={doSomething} />`           | **Controlled (by parent component):**: You want to be fully in charge of this component. You probably want to do this when some requirement of the feature wants the parent component to be able to add some customized behaviour for the component.                                                                                                                                            |
| `<input defaultValue="John Doe" />`                        | **Uncontrolled with a desired starting value**: Use `defaultValue` when you want to "guide" the component but without making it controlled and have React managing the state. A lot of really simple components like Tabs could use this one. You want the tabs to be uncontrolled and handle the logic of which tab is the active tab, but you want to say start on the second tab by default. |
| `<input defaultValue="John Doe" onChange={doSomething} />` | **Uncontrolled with side effects and defaultValue**: Similar to the case above, we'd want this case just to listen in on the uncontrolled component and do some side effect when the data changes while providing the default value to start from.                                                                                                                                              |
