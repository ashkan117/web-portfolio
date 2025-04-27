---
title: "Elixir Clause Anti Patterns"
date: 2025-03-03T20:58:04Z
draft: true
---

```elixir
  defp generate_instances(
         _user,
         %{recurrence_rule: [%RecurrenceRule{frequency: :daily, interval: interval, start_date: start_date}]} =
           template_todo,
         _current_datetime
       )
       when start_date != nil do
    now = DateTime.utc_now()
    due_dates = DateUtils.generate_daily_instances(start_date, interval, 10)

    for due_date <- due_dates do
      TodoInstance.create!(%{
        todo_id: template_todo.id,
        due_date_at: due_date
      })
    end
  end

  defp generate_instances(
         _user,
         %{recurrence_rule: [%RecurrenceRule{frequency: :daily, interval: interval}]} = template_todo,
         current_datetime
       ) do
    due_dates = DateUtils.generate_daily_instances(current_datetime, interval, 10)

    for due_date <- due_dates do
      TodoInstance.create!(%{
        todo_id: template_todo.id,
        due_date_at: due_date
      })
    end
  end
```

```

```
