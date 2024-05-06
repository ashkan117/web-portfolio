---
title: "Ecto Associations: Inserting Data to an Existing Parent"
date: 2024-05-06T10:35:23+01:00
draft: true
---

## TLDR

Associations in Ecto are a little confusing. This post will focus on some important quirks that working with associations comes with.
Note that part of the confusion is when to use which flavor of association functions like `put_assoc`, `cast_assoc`, `build_assoc`. Another part 
will focus on that but this post will use `cast_assoc` as the foundation to first clarify things that all these methods have in common.

## Associations as a Whole: All or Nothing

One thing that might not be intuitive at first is that associations work on the whole collection of data. 
Meaning that if you're not careful you can actually replace your associations so read carefully. Inserting one new item might mean actually removing all others!

Let's look at a common scenario that you'll run into; inserting new items to an existing parent item. We'll follow an example that use used in the [Ecto docs](https://hexdocs.pm/ecto/Ecto.Changeset.html#put_assoc/4-example-adding-a-comment-to-a-post).

```elixir
comment_one = %Comment{body: "Oh really?"}
comment_two = %Comment{body: "Read the Friendly Manual!"}
post
|> Ecto.Changeset.cast_assoc(:comments, [comment_one, comment_two])
|> Repo.update!()
```

At first this looks like a reasonable way to insert two new comments but this would actually replace all the previous comments with just these two. So whenever you see in the docs that methods like `cast_assoc` and `put_assoc` work with the **full data** or the **work with the associations as a whole** this is what that means.

## Preload + List Manipulations

Now that we see that we work with the whole data how can we handle this? By fetching the data. That is Why
you'll commonly see a preload tied with `cast_assoc` like changes. Since we're working with the whole data 
we need to think of this update operation as a list operation. If you have an existing list of comments 
and some new comments then you need to update the list with prepend or the `++/2` operation.

```elixir
post
|> Repo.preload(:comments)
|> Ecto.Changeset.change()
|> Ecto.Changeset.put_assoc(:comments, [comment_one | [comment_two | post.comments])
|> Repo.update!()
```

## Reversing the Relationship (Update -> Insert): A more efficient approach

The final part of this post will cover a better way to handle this scenario. Which the docs highlight as well. If you think about these operations you can run into many situations where this approach won't be too efficient. Think of `has_many` relationships where you would have large lists. A Discord Chat might have many messages in each channel. A flashcard app might have hundreds of cards per Deck. Loading the full list of child elements just to update a few could have its problems. Therefore switch the relationship around. 

Another view of this is that we want to insert a couple comments. As long as we update the proper relationships in the child element then on a future load of all of the comments, things will work.

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

It's important to tie this back to what the database operations are. In reality this is just ensuring that foreign key relationship is maintained. In this case the comment database has a `post_id` column that is updated.

```elixir
%Comment{body: "simple example", post_id: post.id}
|> Repo.insert!()
```

The next time we load all the comments of the post, it will find this new comment.


