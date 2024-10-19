---
title: "Ecto Associations II: Updating Associations as a Whole"
date: 2024-05-06T11:26:41+01:00
draft: true
---

## Introduction

In the [previous post](/posts/ecto-associations-cast-assoc/), we focused on
a common use case when working with associations.
Inserting new elements to an existing parent field. This covered concepts like
why we need to preload the associated data in these operations. In this post,
we'll cover another example although probably less common case to explore how
Ecto is handling things in greater detail. In this case we'll cover bulk
updates/inserts. In my todo app, I want the ability to update events while
being able to insert new ones. One reason you might want to do this as a bulk
action is since the changes are connected to each other. Let's say I am scheduling
my day and then an important meeting pops up. In this scenario, I would want
to in a single transaction

1. Update the existing events in my calendar to make room for the new meeting
2. Create a new event for the unexpected event

In this scenario then, you'd want to work with the whole data all at once instead.

## Exploring the scenario

So in this case let's imagine the situation is that this important event popped up.
A nice user experience might be that they delete an existing event since there really
is no time for it anymore since the new event takes priority. They don't want
to update the other events to "get out of the way" so we just have some checkbox
or dialog that shows up notifiying them that this will move the existing
events out of the way.

Under the hood we'd transform the user input to the following edit:

1. Delete the event
2. Update the existing events that conflict
3. Create the new emergency event

## Understanding the `cast_assoc` algorithm

### The Scene

In reality its convenient to view the `cast_assoc` like functions as algorithm
working on a list of data. Step 1 is to first load all the events tied to today.

```elixir
Agenda
|> Repo.get!(id)
|> Repo.preload(:events)

# returns the following existing events
[
%Event{id: 123, name: "This will be deleted"},
%Event{id: 124, name: "This will be pushed up"}
]
```

The data that we might receive is the following.

```elixir
%{"id" => 22, "name" => "Today's Agenda", "events" => [
  # move the existing event to a new time
  %{"id" => 123, "start" => "2024-05-06T10:00:00Z", "end" => "2024-05-06T12:00:00Z"},
  # the new event will be
  %{"start" => "2024-05-06T08:00:00Z", "end" => "2024-05-06T10:00:00Z"},
]}
```

> Note that the new event does not contain an id. It does not yet exist in the
> database so there's no way it could.

### Breaking Down the Scene

Take some time to look at the existing events that the preload data gets back
as well as the data that the request will receive. If we were the ones trying
to implement the Ecto ability to update a collection as a whole what cases
would need to consider? The more you think about it, the more you realize
that it's not the most straightforward thing. You'd need to determine:

- How can we tell that an item is a **new** element aka an database **insertion**
- How can we tell that an item is a **existing** element that should be replaced
  aka an database **replace**
- How can we tell that an item is a **existing** element that should be updated
  aka an database **update**

> Replace can be thought of as a delete-like strategy. Some cases you don't
> want to delete the child element, you'd want to keep the item in your
> database but in terms of the current association. In this example,
> we could do either. We could altogether delete the Event since it won't
> actually be used. We could also keep it in our database to gain insights
> for our user. Maybe we could look at which times were the Events that
> were not used, and maybe we see a pattern that around certain times
> more "urgent" events pop up. If interested please look at the post focused on this.

## Difference Algorithm

We can finally work towards proving the claim that viewing these operations
as an algorithm is helpful. This algorithm is really a difference algorithm. We have

- Initial State: The list from Preload
- Changes: The new list we get from the HTTP request
- Output: A list of action's to perform (Updates, Deletes, Inserts)

Initially we have two events, in the changes we received from the HTTP request we
have two events as well. Yet, as we stated in the beginning there are three updates.
One delete, one update, and one insert.
I think at first it sounds difficult to create an algorithm smart enough
to know that but it's not actually too bad.

The ID plays a key role. Note that all items in the preload will have an ID
since it's coming from the database. It's guaranteed to have a unique ID if
it's the primary key. The changes that we received are not guaranteed to always
have IDs. This is because new data will not have any ID since the data has not
yet been persisted.

> The ID in combination with the difference of the initial list and change list
> is actually all you need to determine the action.

### Intuitive Solution

Let's think of this algorithm as making three passes. The order doesn't really matter

- Updates: Search the changes list for IDs to determine the updates
- Inserts: Search through the changes list for items with no IDs.
- Deletes: Find which IDs exist in the initial list but not the changes list.

> Note: The following list is not 100% complete. There is a slight modification
> to this algorithm but I think this is the intuitive model. There is
> one edge case to this that we'll discuss later. For reference I'll
> put what the initial data and changes are.

```elixir
# initial list
[
%Event{id: 123, name: "This will be deleted"},
%Event{id: 124, name: "This will be pushed up"}
]

# changes list
[
  # move the existing event to a new time
  %{"id" => 123, "start" => "2024-05-06T10:00:00Z", "end" => "2024-05-06T12:00:00Z"},
  # the new event will be
  %{"start" => "2024-05-06T08:00:00Z", "end" => "2024-05-06T10:00:00Z"},
]

```

### Creating the List of Actions

This is pretty straightforward, looking at just the changes list we look for IDs.
Any IDs that are present we can consider as an update. In our example we see
that the element with id 123 exists.

Next comes insertions. Looking at the changes list we see that the second element
doesn't contain any ID. Therefore, we consider this a new item and
we'll insert it into the database.

Finally comes something missing, our deletes. In the changes list we're actually
missing an ID that exists in our initial list. There is no element with ID 124.
Since we're working with the data as a whole this is interpreted as a delete.

```elixir
# pseudo code
[
  {type: update, action: "Update element with ID 123" },
  {type: insert, action: "A new item with the params of element 2 in the changes list" },
  {type: delete, action: "Delete element with ID 124" },
]
```

## Ammendamt

If you understand this you're 90% there. Along the way I took some
small liberties to keep things as simple as possible.

### Insertions

There are a couple cases that the changes list can count as an
insert that I did not mention.

1. What if there is an element in the change list that includes an ID, but
   one that does not exist in the initial list.
2. Edge Case: What if an element provides an ID with the value `nil`

The first case is alot more important than the second. Let's say that you want the
client to be able to determine the ID. Maybe you're using UUIDs which means that
the client could generate the ID. Or maybe the ID is timebased? In any case,
in the situation that an ID is provided that does not exist in our
Database it does make sense that this case is an insert.

The second case is more of an edge case, if the changes list includes an item
that includes a field id but that value is **strictly** `nil`
that also counts as an insert.

## Deletes are Actually Replaces

This ammendment is an important one and one I'm considering to potentially change.

Ecto considers the situation where
