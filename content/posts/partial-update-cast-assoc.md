---
title: "Partial Update Cast Assoc"
date: 2025-03-28T09:36:01Z
draft: true
---

```elixir

   message_template_with_preloads = Repo.preload(message_template, preloads)

    attrs_with_merged_metadata =
      if Map.has_key?(attrs, "template_metadata") do
        current_metadata = message_template_with_preloads.template_metadata || %{}
        new_metadata = attrs["template_metadata"]
        merged_metadata = Map.merge(current_metadata, new_metadata)
        Map.put(attrs, "template_metadata", merged_metadata)
      else
        attrs
      end

    message_template_with_preloads
    |> MessageTemplate.changeset(attrs_with_merged_metadata)
```

- Outside of the changeset
- Seems like something Ecto.Changesets can handle

cast_assos with field
If the parameter contains an ID and there is an associated child with such ID, the parameter data will be passed to MyApp.Address.changeset/2 with the existing struct and become an **update operation**
