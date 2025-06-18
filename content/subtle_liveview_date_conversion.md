---
title: "Subtle_liveview_date_conversion"
date: 2025-06-07T14:34:38+01:00
draft: true
---

## New Bug

In my application we have a calendar component that displays the list of events.
For some reason, the list of events started to correctly show the events but
then something would happen then everything would disappear.

I was stumped. I couldn't reproduce the issue locally and I wasn't sure
which aspect was the issue.

Was it a GoogleCalendar API call issue, was in the JS world with FullCalendar,
was I doing something wrong in the backend with LiveView or Elixir?

## Debugging

I wasn't entirely sure what would be causing this. The flow of the information
would be FullCalendar would trigger the "dates_changed" which would simply
enough send that information to a LiveComponent.

```js
datesSet: (dateInfo) => {
  this.pushEventTo(this.el, "dates_changed", dateInfo);
};
```

In the LiveComponent, I would then have the following code to get the
dates and work with them

```elixir
  def handle_event("dates_changed", %{"start" => start_iso, "end" => end_iso}, socket) do
    start_dt = start_iso |> DateTime.from_iso8601() |> elem(1)
    end_dt = end_iso |> DateTime.from_iso8601() |> elem(1)
    ...
    {:noreply, socket}
  end
```

## THe Issue

The issue was subtle, the `start` and `end` values in the `dateInfo` where JS Dates.
When we were passing this information back to the LiveComponent, LiveView
comes in and translates the JS Dates into Elixir DateTimes.

```bash
start: Date Sat Jun 07 2025 00:00:00 GMT+0100 (British Summer Time)
end: Date Sun Jun 08 2025 00:00:00 GMT+0100 (British Summer Time)
```

But Elixir received it as

```bash
[info] start date: ~U[2025-06-06 23:00:00.000Z]
[info] end date: ~U[2025-06-07 23:00:00.000Z]
```

Now putting this all together, the reason why it was removing all the events
was that we were showing just the day mode. So although the events list
under the hood was not empty, since it was getting the wrong dates
and we were displaying only todays events, it would always show an empty
today view. This made me take longer to realize that the dates were off
since I saw that dates coming in so I thought it was a render issue.

## Solution

I'm still not entirely sure why the Date translation wasn't being received
properly but it seems like FullCalendar might not be a stranger to this.
In the dateInfo they also contain `startStr` and `endStr` which are the
ISO8601 strings of the dates.

The reason why strings are nice in this case is that no conversion is needed.
So we can get the userTimeZone from the Intl API and then shift the dates
into the user's timezone.

```js
datesSet: (dateInfo) => {
  let userTz = Intl.DateTimeFormat().resolvedOptions().timeZone;
  this.pushEventTo(this.el, "dates_changed", {
    startStr: dateInfo.start.toISOString(),
    endStr: dateInfo.end.toISOString(),
    userTz: userTz,
  });
};
```

```elixir
  def handle_event("dates_changed", %{"startStr" => start_iso, "endStr" => end_iso, "userTz" => user_tz}, socket) do
    {:ok, start_utc, _} = DateTime.from_iso8601(start_iso)
    {:ok, end_utc, _} = DateTime.from_iso8601(end_iso)

    # 2) Shift that UTC instant into the user’s zone
    start_dt = DateTime.shift_zone!(start_utc, user_tz)
    end_dt = DateTime.shift_zone!(end_utc, user_tz)
    ...
    {:noreply, socket}
  end
```
