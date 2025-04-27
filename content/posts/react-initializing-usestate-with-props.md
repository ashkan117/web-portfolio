---
title: "React Initializing Usestate With Props"
date: 2025-04-27T13:47:20+01:00
draft: true
---

## Required Reading

This article will cover the relationships you might run into

First and foremost, please go read the lovely series that TKDodo
created on common pitfalls. This article specifically covers
the props and useState relationship he covers in [putting props to useState](https://tkdodo.eu/blog/putting-props-to-use-state).

What I'll be covering is what patterns or parent child component relationships tend
to create this issue but first let's highlight the main issue. **This tends
to happen when the child component is "smart"**. This means that we're
trying to abstract away some work into this component.

## useState's initialState is determined and fixed on mount (first render)

When you use useState, you're essentially creating a state variable
that is tied to the component. When you define the `initialState` of the
state then this means that the value is locked in. This might seem like
an obvious thing and in some cases it is.

```jsx
const [count, setCount] = useState(0);
```

In the case above this is obvious that the we start from 0 and move on there.
The `initialState` starts at 0 and is fixed. So in what cases might we provide
an `initialState` that would change?

### Asynchronous Props

In one of the cases I ran into this issue is that the child component is a "controlled"
component but the props passed in are determined asynchronously. Let's say you
have a list of tags for some todo application that you want to display
through a multi-select. Sounds simple enough, we should then

1. Fetch the tags
2. Pass the tags as props

What could possibly go wrong here?

Well the issue was that the component I was using a component that used `useState`
to manage the label of the component. This was because in this component, if
the user selected 20 tags, we want to show maybe 2 of those names then have
+18 more.

So how does the React component look like? Well it was designed as an
uncontrolled component so it would have a `selected` prop that would
be an array of strings. We also had an `options` props that would represent
the list of labels and values that the select components typically have.

From that prop then we'd determine the `selectedLabels`.

```jsx
useEffect(() => {
  setSelectedLabels(calculateResult(selected, options));
}, [selected]);

const [selectedLabels, setSelectedLabels] = useState(
  calculateResult(selected, options), //determined initial state from the props
);
```

This an anti-pattern as TKDodo mentions in [this part](https://tkdodo.eu/blog/putting-props-to-use-state#the-non-solution)
of the article. Even worse in my case this

- We're missing the `options` prop inside the dependency array
- The options was being determined by props that were being calculated asynchronously

### Async Nature

The `options` being passed in was initially empty since we haven't fetched
the list of tags to present yet. Eventually in the parent component would load
the list of tags and from that create the list of `options` that we would pass
into the child component.

```jsx
const [options, setOptions] = useState([]);

useEffect(() => {
  fetch("/api/tags")
    .then((res) => res.json())
    .then((data) => {
      data.map((tag) => {
        setOptions((prev) => [...prev, { label: tag.name, value: tag.id }]);
      });
    });
},

return (
  ...
  <MultiSelect options={options} onChange={handleChange} />
)
```

The issue is that once the `MultiSelect` component is mounted, it
already received the `initialState` of `[]` and it went on to initialize
the label of the component to an empty array.

Once the tags come in, the props are updated but the MultiSelect component
is already mounted. The `initialState` calculation won't rerun.

## What do we do then?

There are many ways to solve this problem but the simplest in my case
was to not use `useState` to begin with. The state could be purely be **computed**
from the props so we never needed useState. That anti-pattern is usually
a code smell that should cause you to think "should I be using useState?"

So the `MultiSelect` component was "smart". It has state that it manages and
therefore uses `useState`. The parent component got its data asynchronously
and then passed it into the child component but since the child component
used the asyncrounous props to determine the `initialState` we ran
into problems.

TKDodo goes over multiple solutions in [his putting props to use state article](https://tkdodo.eu/blog/putting-props-to-use-state)
however I want to just [conditional render solution](https://tkdodo.eu/blog/putting-props-to-use-state#1-conditionally-render-the-detailview).

In my case, if we really had to keep `useState` then you have to consider
should we render this component to begin with?

### Loading State (Probably?)

Can we add some loading state component that renders while we wait for
the data? Once the data is there, we can now render the multi-select
component **for the first time**.

```jsx
const [options, setOptions] = useState([]);
const [isLoading, setIsLoading] = useState(true);

useEffect(() => {
  fetch("/api/tags")
    .then((res) => res.json())
    .then((data) => {
      data.map((tag) => {
        setOptions((prev) => [...prev, { label: tag.name, value: tag.id }]);
      });
    });
},

return (
...
  {isLoading && <Loading />}
  (!isLoading && <MultiSelect options={options} onChange={handleChange} />)
)
```

Since the `MultiSelect` component is now rendered conditionally, it ensures
that the `initialState` is determined at the right point which
in our case is when the data is loaded.

### Keys? (Probably Not?)

I don't think there's a great key we can use in this case for a list
of items so I don't think this would be a great solution.
