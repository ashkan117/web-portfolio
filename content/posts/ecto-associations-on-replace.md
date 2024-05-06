---
title: "Ecto Associations on Replace"
date: 2024-05-06T12:31:47+01:00
draft: true
---

## Introduction

This posts focuses on Ecto's `on_replace` option that is relevant when working with associations. We'll break down when it becomes relevant and explore the options that show up. We can think of this option as defining a strategy to take when associations are severed.

- Temporary Ownership


## Reminder: Ecto Associations are All or Nothing
Remember that association methods like `cast_assoc` work with the associations as a whole. This means we must view the data being placed in those methods as the desired value for the whole association. This is a common misconception in the beginning when working with Ecto. For example the following case as the documentation is the **wrong way to insert a new comment to a blog post**.

```elixir
post
|> Ecto.Changeset.change()
|> Ecto.Changeset.put_assoc(:comments, [%Comment{body: "bad example!"}])
|> Repo.update!()
```

> The reason why the example above is wrong is because put_assoc/4 always works with the full data. So the example above will effectively erase all previous comments and only keep the comment you are currently adding. Instead, you could try - Ecto Docs

## Warning

Note that the documentation gives us extra warnings when using the `:delete` or `:delete_if_exists` options. It's bad enough if you use `put_assoc` or the like to wrongly. Like we mentioned you tried to insert an additional element into the list and instead, you replaced the whole list with a single element. Delete and Delete if Exists will actually delete the data behind the associations. Simply put: 
- `put_assoc` like functions mistakes result in the loss of data for a single Parent Schema
- `delete` or `:delete_if_exists` like functions mistakes result in the loss of data for ALL Parents relationships dependent on the same child elements


## When is something replaced
In the [docs](https://hexdocs.pm/ecto/Ecto.Changeset.html#module-the-on_replace-option) it states: "When using any of those APIs, you may run into situations where Ecto sees data is being replaced. For example, imagine a Post has many Comments where the comments have IDs 1, 2 and 3. If you call cast_assoc/3 passing only the IDs 1 and 2, Ecto will consider 3 is being "replaced" and it will raise by default."

Essentially we'll look at the initial list and see 1,2,3. We look at the final state and see [1,2]. What should we do with the element with ID 3? In the world of Ecto we'll consider this element replaced!

## Why Is this Needed?

Before we explore the replace strategies, I just want to highlight why this is even necessary. Ultimately, it's tied to understanding that the changes we want to do with this associations is not what we necessarily want done to our database.

Let's take the docs example of a post initially having comments with `[1,2,3]` and later with `[1,2]`. As the application developer you might want to just consider this as a delete. Maybe the user deletes their comment, the frontend removes this item from the list and then makes a request to the backend that the new list of comments should just be `[1,2]`. 

However, from the standpoint of the application as a whole we might actually want this data to still exist. If we just delete the element from the database then we can't use this information. Maybe a moderator deleted this comment. It's reasonable that if a user has their comment deleted enough from moderators that we block that user or if we have the bandwidth to set some internal flag that this user needs to be monitored. Therefore, this operation is not a delete from the database, its moreso updating the associations so that the post no longer shows this comment.

## What does that mean?

Hopefully why it's needed is a little more clear. Now we'll go through the allowed values.

### Raise (default)

:raise (default) - do not allow removing association or embedded data via **parent changesets**

This just means that by default you need to explicitly state your policy. If you try to make changes without defining this `on_replace` option we won't let you. Note that this also says via parent changesets. This just means that you can delete the element from the child point of view. 

```elixir
Repo.delete(comment)
```

### Mark as Invalid

:mark_as_invalid - if attempting to remove the association or embedded data via parent changeset - an error will be added to the parent changeset, and it will be marked as invalid

This value is only slightly less strict than the `:raise` option. Instead of throwing an error, we just consider the changeset invalid.

When might you want this? You're explicitly not allowing removal of this association. So once something is created it cannot be removed. I haven't used this but according to ChatGPT a useful case could be in an Book and Authors relationship. Once a book is created the authors would never change so you can explicitly deny any
changeset that attempts to delete the association.

### Nilify (Delete in the eyes of the association ONLY)
:nilify - sets owner reference column to nil (available only for associations). Use this on a belongs_to column to allow the association to be cleared out so that it can be set to a new value. Will set action on associated changesets to :replace

This is a common scenario. Think of something with temporary ownership for example. Maybe you're making a Car Rental service app. One specific car could have many people use it throughout the years. We wouldn't want to just delete the information of people since Car History is crucial. However, there are moments where no one is currently renting a car.

```elixir
 def return_car(car_id) do
    car = Repo.get!(Car, car_id)

    changeset = Car.changeset(car, %{current_owner_id: nil})

    case Repo.update(changeset) do
      {:ok, _car} ->
        {:ok, "Car has been successfully returned"}
      
      {:error, changeset} ->
        {:error, "Could not return the car", changeset}
    end
  end
```

In this case we want the changeset to be considered valid which eliminates our previous values `:raise` and `:mark_as_invalid`.

### Update

:update - updates the association, available only for has_one, belongs_to and embeds_one. This option will update all the fields given to the changeset including the id for the association

With closely tied data, update might make more sense. An (ChatGPT) example of this might be Orders and LineItems. 
```elixir
  schema "orders" do
    ...
    has_many :line_items, LineItem, on_replace: :update
  end

  schema "line_items" do
    field :product_name, :string
    field :quantity, :integer
    field :price, :decimal

    belongs_to :order, MyApp.Order
    ...
  end
```

In this case the data is tied closely enough where if a line item is changed, then we would maybe just want to update that specific line item, instead of deleting and creating a new item.

In order to make things more clear lets compare and contrast things with nilify. In both these cases we were able to result in a valid changeset. `:raise` and `:mark_as_invalid` both prevented this. The key difference is that `:nilify` does not "touch" the association whereas `:update` does. 

### Delete

This one is pretty straight forward. In some cases you really don't care about the associated item or you just want to delete things so that our database does not get flooded. You could think of a person deleting their account. Let's say the data is not too important or you want to value their data privacy. You actually want to delete the associations.


### Delete If Exists (Race Condition)
In some cases where multiple independent people could delete the same element we'd run into a race condition. User One makes a request which deletes the element. User Two makes a request at a slightly later time trying to delete an element just marked as deleted. Ecto will complain with `Ecto.StaleEntryError`. This is a more specific flavor of delete that prevents this. This is only relevant again in a situation where something could be requested to be deleted multiple times. 

Why shouldn't we always use it.




- This does not apply to embedded data

