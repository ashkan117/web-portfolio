---
title: "Ecto Gotachas Structs on Not Change Tracked"
date: 2024-05-27T10:03:54+01:00
draft: true
---

## Highlighting the Problem
While builing out my side project, I wanted to really leverage and learn more about Ecto. One of the things I was working with was managing a list using Changesets. The struct in question is a simple one called `Interval` which logically just represents a start and end datetime.

I was working on a feature that would allow multiple Intervals to be shifted by some fixed number of `minutes`. So given one interval that represents 8-830am and another for 830am - 9am, and 30 mintues. This would reults in 830am-9am and 9am-930am. Simple enough right.

What if I were to tell you that I the following code would not properly determine that the intervals changed?
```elixir
    changes =
      intervals_to_change
      |> Enum.reduce([], fn interval, interval_acc ->
        new_start = DateTime.add(interval.start, minutes, :minute)
        new_end = DateTime.add(interval.end, minutes, :minute)
        [%Interval{interval | start: new_start, end: new_end} | interval_acc]
      end)

    Ecto.Changeset.put_embed(changeset, :intervals, changes)
```

Yet this results in no changes being shown in the changeset
```elixir

#Ecto.Changeset<action: nil, changes: %{}, errors: [], data: #Alpen.Intervals.IntervalList<>, valid?: true>
```

The fix looks identical, feel free to play a game of spot the difference in the meantime.
```elixir
    changes =
      intervals_to_change
      |> Enum.reduce([], fn interval, interval_acc ->
        new_start = DateTime.add(interval.start, minutes, :minute)
        new_end = DateTime.add(interval.end, minutes, :minute)
        [%{id: interval.id, start: new_start, end: new_end} | interval_acc]
      end)
    Ecto.Changeset.put_embed(changeset, :intervals, changes)
```

```elixir
#Ecto.Changeset<
    action: nil,
    changes: %{
      intervals: [
        #Ecto.Changeset<
          action: :update,
          changes: %{
            start: ~U[2024-05-24 08:30:00Z],
            end: ~U[2024-05-24 09:00:00Z]
          },
          errors: [],
          data: #Alpen.Intervals.Interval<>,
          valid?: true
        >,
        #Ecto.Changeset<
          action: :update,
          changes: %{
            start: ~U[2024-05-24 09:00:00Z],
            end: ~U[2024-05-24 09:30:00Z]
          },
          errors: [],
          data: #Alpen.Intervals.Interval<>,
          valid?: true
        >
      ]
    },
    errors: [],
    data: #Alpen.Intervals.IntervalList<>,
    valid?: true
>
```

## Structs are not Change Tracked

Confused I took a look through the docs and found this section on [put_assoc/3](https://hexdocs.pm/ecto/Ecto.Changeset.html#put_assoc/4). Specifically look at the section were it talks about when the associated data is a struct.

> Different to passing changesets, structs are not change tracked in any fashion. In other words, if you change a comment struct and give it to put_assoc/4, the updates in the struct won't be persisted. You must use changesets instead.




