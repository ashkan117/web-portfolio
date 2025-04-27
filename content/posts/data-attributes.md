---
title: "Data Attributes"
date: 2025-04-21T14:51:06+01:00
draft: false
---

## Framework/Library or Native? Convention or Specification?

In the development world there is so many frameworks and libraries that build on
abstractions on top of other abstractions it gets hard to understand is this
thing that I'm doing specific to the framework or library that I am using
or is it something else.

I ran into this recently when building out my todo app. I was building this
with Phoenix Liveview and using a few JS libraries like
SortableJS and FullCalendar JS and noticed me using html attributes like `data-`.

I found myself wondering where is this coming and dug deeper to find that
this is just part of the HTML Spec called [custom data attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/data-*).

I think the definition on Mozilla Docs do a good job.

> The data-\* global attributes form a class of attributes called custom data
> attributes, that allow proprietary information to be exchanged between the
> HTML and its DOM representation by scripts.

I think we can think the data attributes as two main things

1. A promise that this is not HTML specific, HTML will not use this
   to do anything
2. Your JS wants to do something based off HTML.

## Explaining the Setup

Now to explain my scenario better I was dynamically adding some custom data
attributes to an html div. In my application you can add a Project or a more
concrete Todo to the calendar.

So from the LiveView world I needed to send that information down to
JS through HTML. I did so with the custom attribute `data-type` which could be
a `:project` or a `:type`.

Now in the JS world how should I get this information? HTML provides a set
of builtin tools like [`dataset`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset)
that make things like this easy to work with in JS world.

- Treats all the custom data attributes (all attributes prefixed with `data-`)
  as a simple object.
- Handles the converting hyphenated names (HTML world) to camelCased names (JS world)
  - "Hyphenated names become camel-cased. For example, data-foo-bar="" becomes element.dataset.fooBar."

```js
const type = externalEventDropInfo.draggedEl.dataset.type;
if (type === "project") {
  // style rules for project
} else {
  // style rules for todo
}
```

## Conclusion

Web frameworks like React and LiveView in a really simple way control what is
being rendered and in many situations you're adding in some HTML dynamically.
However there are some cases where the state you're storing in your frontend
framework of choice has no way to be shared to a JS library you want to use.
In my case LiveView provides "hooks" but to transfer that state to a JS library
I needed to do so though these `data-*` attributes so that
I can later determine what classes I want to use.

I wanted to uncover for myself but hopefully for others that there are builtin
mechanisms that already exist and help you do things simply. No library or magic
needed.

## Extra

[Here is](https://www.youtube.com/watch?v=XtEs0SZ_4Y0&t=15s) a
nice practical video that
you can watch that highlights a similar use case where using plain
HTML and JS you can add data attributes to an element and then
using JS you can control what is being rendered based off those attributes.
