---
title: "Ecto Associations I: Working with Associations as a Whole"
date: 2024-05-06T10:35:23+01:00
draft: false
---

## Motivation

One thing that might not be intuitive at first is that associations work
on the whole collection of data. You'll see this mentioned in a few ways in
the docs. They'll usually say that methods like `cast_assoc` and `put_assoc`
work with the **full data** or the **work with the associations as a whole**.
Let's look at an example to see what this means.

## Work with the Associations as a Whole

`post_assoc` looks like `put_assoc(changeset, name, value, opts \\ [])`.
When it says that we must with the **data as a whole** that means that
the value that we pass represents the ultimate end value of the association.
So the following code snippet.

```elixir
post
|> Repo.preload(:tags)
|> Ecto.Changeset.change()
# the tags field will end up being 2 items
|> Ecto.Changeset.put_assoc(:tags, [existing_tag, new_tag])
|> Repo.update!()
```

## Associations must have their previous relationships maintained (Need for `preload`)

Working with the data as a whole tells us what the final result should be
but there's some work that needs to happen before that. For example,
in the previous case, we're ending with two tags with that post.
Let's say one of them exists and the other is a new tag that we're creating.
At the very least we need to.

1. Create a new Tag.
2. Make a note that the post is connected to the existing tag as well.

There's one other potential case. **What if previously the post was connected
to other tags?** Let's say previously tags was equal to `[old_tag, existing_tag]`.
If the ending value will be [existing_tag, new_tag] that means that we need
to make sure that the `old_tag` is no longer tied to `post`.

> For this reason we are required to preload the associations. We always
> need to make sure that we properly maintain **previous** relationships.

## Warning: Be careful not to "append" items to a list

If you're not careful or are just starting you can
actually replace your associations so read carefully. If you're trying to
append a single item to a list of associations you could might mean
actually replace all your data with that item!

Let's look at a common scenario that you'll run into; inserting new items to an existing
parent item. We'll follow an example that use used in the [Ecto docs](https://hexdocs.pm/ecto/Ecto.Changeset.html#put_assoc/4-example-adding-a-comment-to-a-post).

```elixir
new_comment = %Comment{body: "Read the Friendly Manual!"}
post
|> Repo.preload(:comments)
|> Ecto.Changeset.change()
|> Ecto.Changeset.put_assoc(:comments, [new_comment])
|> Repo.update!()
```

At first this looks like a reasonable way to append one new
comments but this would actually replace all the previous comments with just
one. Since `cast_assoc` and `put_assoc` and work with the **full data**
what we're actually doing is saying that the final value for this
post comments is equal to the single list `[new_comment]`.

## Reversing the Relationship For Appends (Update -> Insert): A more efficient approach

Think of `has_many` relationships where you would have large lists. A Discord
Chat might have many messages in each channel. A flashcard app might have hundreds
of cards per Deck. Loading the full list of child elements just to update a few
could have its performance problems. Therefore switch the relationship around.
Another view of this is that we want to insert a couple comments.
As long as we update the
proper relationships in the child element then on a future load of all of
the comments, things will work.

```elixir
%Comment{body: "better example"}
|> Ecto.Changeset.change()
|> Ecto.Changeset.put_assoc(:post, post)
|> Repo.insert!()

%Comment{body: "Thank you for reading the friendly manual!"}
|> Ecto.Changeset.change()
|> Ecto.Changeset.put_assoc(:post, post)
|> Repo.insert!()
```

It's important to tie this back to what the database operations are.
In reality this is just ensuring that foreign key relationship is maintained.
In this case the comment database has a `post_id` column that is updated.

```elixir
%Comment{body: "simple example", post_id: post.id}
|> Repo.insert!()
```

> If you don’t preload the :tags association, Ecto will raise an error
> because it does not have enough information about the
> existing tags to properly update it (e.g., which records to add, remove, or modify).
>
> ```bash
> ** (ArgumentError) you are attempting to change relation :tags of SampleApp.Post,
> but the data is not loaded. Please preload your associations before manipulating
> them through changesets.
> ```

## Conclusion

- We must work with the **full data** / **work with the associations as a whole**
- In order to properly maintain SQL relationships we must know what the
  previous state was with `preload`
- Since we must work with the full data, be careful for performance issues
  of how we update list relationships
