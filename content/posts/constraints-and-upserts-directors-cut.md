---
title: "Constraints and Upserts Directors Cut"
date: 2024-03-08T16:44:04Z
draft: true
---


## Motivation
Going through the [Constraints and Upserts](https://hexdocs.pm/ecto/constraints-and-upserts.html#upserts-and-insert_all) guide helped me learn a ton of great details however there were 
a few gaps in my understanding and a few points where they stated something that I did not fully understand. Therefore, 
this article goes through the same guide but will go slightly deeper in areas that I did not get until playing with the code.
So if you haven't gone through the original article, go do that now. The first article was fairly involved for me so 
if it's your first run through don't bother reading this one until a few days after or a week after so that things can marinate. 
This article picks up at the [upsert section](https://hexdocs.pm/ecto/constraints-and-upserts.html#upserts) where it first introduces the 
`on_conflict` option.

## Upserts with insert/2
So we just learned that in order for upserts to be possible we need to add the `on_conflict` option. We start with the `:nothing` value.
``` elixir
defp get_or_insert_tag(name) do
  Repo.insert!(
    %MyApp.Tag{name: name},
    on_conflict: :nothing
  )
end
```

Let's say we wanted to create a Post with a new tag called "travel". Ultimately once we reach the `get_or_insert_tag` it will create/insert the tag 
since it's new. Sweet! However, if you run it again you'll notice that the Tag struct returned has no id.

``` bash
# I made the get_or_insert_tag public to run it in iex
iex(1)> MyApp.Post.get_or_insert_tag("newlo")

16:42:38.873 [debug] QUERY OK source="tags" db=7.5ms decode=3.1ms queue=2.2ms idle=1779.5ms
INSERT INTO "tags" ("name","inserted_at","updated_at") VALUES ($1,$2,$3) ON CONFLICT DO NOTHING RETURNING "id" ["newlo", ~N[2024-03-08 16:42:38], ~N[2024-03-08 16:42:38]]
%MyApp.Tag{
  __meta__: #Ecto.Schema.Metadata<:loaded, "tags">,
  id: 1,
  name: "newlo",
  inserted_at: ~N[2024-03-08 16:42:38],
  updated_at: ~N[2024-03-08 16:42:38]
}
iex(2)> MyApp.Post.get_or_insert_tag("newlo")

16:42:41.363 [debug] QUERY OK source="tags" db=1.2ms queue=0.1ms idle=1305.9ms
INSERT INTO "tags" ("name","inserted_at","updated_at") VALUES ($1,$2,$3) ON CONFLICT DO NOTHING RETURNING "id" ["newlo", ~N[2024-03-08 16:42:41], ~N[2024-03-08 16:42:41]]
%MyApp.Tag{
  __meta__: #Ecto.Schema.Metadata<:loaded, "tags">,
  id: nil,
  name: "newlo",
  inserted_at: ~N[2024-03-08 16:42:41],
  updated_at: ~N[2024-03-08 16:42:41]
}
````

In this case notice that the item is not actually inserted. It just returns the Tag struct that it mapped to but nothing was persisted.
That lack of interaction between the database is shown through the `nil` value for `id`.

## Why is an issue that the id is nil
The issue to me was not obvious at first. If you think of things in terms of Tags, things are working as expected.
A tag is inserted if it does not exist and not inserted if it does not exist. However, if you think of things in terms of a 
post trying to be inserted there's an issue. Let's insert three new tags when creating a post.

```bash
iex(5)> MyApp.Post.insert(%{"title" => "Test", "body" => "test body", "tags" => "elixir, ecto, thing"})
...
%MyApp.Post{
  __meta__: #Ecto.Schema.Metadata<:loaded, "posts">,
  id: 1,
  title: "Test",
  body: "test body",
  tags: [
    %MyApp.Tag{
      __meta__: #Ecto.Schema.Metadata<:loaded, "tags">,
      id: 3,
      name: "elixir",
      inserted_at: ~N[2024-03-08 16:56:27],
      updated_at: ~N[2024-03-08 16:56:27]
    },
    %MyApp.Tag{
      __meta__: #Ecto.Schema.Metadata<:loaded, "tags">,
      id: 4,
      name: "ecto",
      inserted_at: ~N[2024-03-08 16:56:27],
      updated_at: ~N[2024-03-08 16:56:27]
    },
    %MyApp.Tag{
      __meta__: #Ecto.Schema.Metadata<:loaded, "tags">,
      id: 5,
      name: "thing",
      inserted_at: ~N[2024-03-08 16:56:27],
      updated_at: ~N[2024-03-08 16:56:27]
    }
  ],
  inserted_at: ~N[2024-03-08 16:56:27],
  updated_at: ~N[2024-03-08 16:56:27]
}
```

Great! We created the post associated with three new tags. However, try running that command again and you'd get an error.
```bash

** (Ecto.NoPrimaryKeyValueError) struct `%MyApp.Tag{__meta__: #Ecto.Schema.Metadata<:loaded, "tags">, id: nil, name: "elixir", inserted_at: ~N[2024-03-08 16:56:33], updated_at: ~N[2024-03-08 16:56:33]}` is missing primary key value
    (ecto 3.11.2) lib/ecto/repo/schema.ex:1015: anonymous fn/3 in Ecto.Repo.Schema.add_pk_filter!/2
    (elixir 1.14.0) lib/enum.ex:2468: Enum."-reduce/3-lists^foldl/2-0-"/3
    (ecto 3.11.2) lib/ecto/repo/schema.ex:424: Ecto.Repo.Schema.do_update/4
    (ecto 3.11.2) lib/ecto/association.ex:1575: Ecto.Association.ManyToMany.on_repo_change/5
    (ecto 3.11.2) lib/ecto/association.ex:648: anonymous fn/8 in Ecto.Association.on_repo_change/7
    (elixir 1.14.0) lib/enum.ex:2468: Enum."-reduce/3-lists^foldl/2-0-"/3
    (ecto 3.11.2) lib/ecto/association.ex:644: Ecto.Association.on_repo_change/7
    iex:6: (file)
```

The reason why is twofold. 
1. When an insert is not successful it will return the inserted entity **minus the id**
2. Inserting an association with an `id` of `nil` will fail.

The first point is surprisingly subtle and I did not know this for a long time. When you insert an element using Ecto it will return everything at the SQL level. 
This makes a lot of sense when you think about it. If you had to insert an item, then query to get it back it would be very inefficient. Therefore, if you can 
get back all the data in one query, that's perfect.

However, in the case where we fail to make an insert (due to the tag names not being unique) nothing is returned. I believe at the Ecto/Elixir level since we have access to the 
fields that are not the id, it will return everything minus the id like before.
```bash
%MyApp.Tag{
  __meta__: #Ecto.Schema.Metadata<:loaded, "tags">,
  id: nil,
  name: "newlo",
  inserted_at: ~N[2024-03-08 16:42:41],
  updated_at: ~N[2024-03-08 16:42:41]
}
```

The reason we need a valid id is that we can associate the tag with the post in our database. We need to be able to create a row like the following. Create a tag
```bash
# create the tag
INSERT INTO "tags" ("name","inserted_at","updated_at") VALUES ($1,$2,$3) ON CONFLICT DO NOTHING RETURNING "id" ["ecto", ~N[2024-03-08 16:56:33], ~N[2024-03-08 16:56:33]]
# create the post
INSERT INTO "posts" ("body","title","inserted_at","updated_at") VALUES ($1,$2,$3,$4) RETURNING "id" ["test body", "Test", ~N[2024-03-08 16:56:33], ~N[2024-03-08 16:56:33]]
# associate the two (this is needed since it's many to many)
INSERT INTO "posts_tags" ("post_id","tag_id") VALUES ($1,$2) [1, 3]
```

The part that becomes an issue in the second time around is the last `INSERT`. We never actually got back the ID so it would be like making the statement.
```bash
INSERT INTO "posts_tags" ("post_id","tag_id") VALUES ($1,$2) [1, nil]
```

## conflict_target to the rescue
The issue is at the SQL level since on a conflict nothing is created, and therefore the id is not returned. So how do we fix things at the SQL level, we need to make this 
an update. By making this an update we're able to change the query into a successful where the "id" could be returned. So we want to be able to say 
"try to insert it, if it fails make it an update and return the fields". Since the insert failed, we know the update will succeed and the id will be returned 
which fixes our earlier problem with `on_conflict: :nothing`. We can then add the following line (this will throw an error in a second)

```elixir
defp get_or_insert_tag(name) do
  Repo.insert!(
    %MyApp.Tag{name: name},
    on_conflict: [set: [name: name]]
  )
end
```

The error pops up telling us that we need another option. 
```bash
** (ArgumentError) the :conflict_target option is required on upserts by PostgreSQL
    (ecto_sql 3.11.1) lib/ecto/adapters/postgres/connection.ex:1890: Ecto.Adapters.Postgres.Connection.error!/2
    (ecto_sql 3.11.1) lib/ecto/adapters/postgres/connection.ex:259: Ecto.Adapters.Postgres.Connection.on_conflict/2
    (ecto_sql 3.11.1) lib/ecto/adapters/postgres/connection.ex:236: Ecto.Adapters.Postgres.Connection.insert/7
    (ecto_sql 3.11.1) lib/ecto/adapters/postgres.ex:132: Ecto.Adapters.Postgres.insert/6
    (ecto 3.11.2) lib/ecto/repo/schema.ex:775: Ecto.Repo.Schema.apply/4
    (ecto 3.11.2) lib/ecto/repo/schema.ex:377: anonymous fn/15 in Ecto.Repo.Schema.do_insert/4
    (ecto 3.11.2) lib/ecto/repo/schema.ex:273: Ecto.Repo.Schema.insert!/4
    iex:11: (file)
```

Finally adding this in will result in a successful insert! Looking at the SQL statements makes the `on_conflict` option look a little less weird. 
If you read the SQL statement then the it's practically the same.

```elixir
defp get_or_insert_tag(name) do
  Repo.insert!(
    %MyApp.Tag{name: name},
    on_conflict: [set: [name: name]],
    conflict_target: :name
  )
end
```
```bash
INSERT INTO "tags" AS t0 ("name","inserted_at","updated_at") VALUES ($1,$2,$3) 
ON CONFLICT ("name") DO UPDATE SET "name" = $4 
RETURNING "id" ["new", ~N[2024-03-08 19:50:25], ~N[2024-03-08 19:50:25], "new"]
```

This finally enables the upsert. In this case I think things look a little weird since when we conflict on the name, we update the name to name it already has. 
The way you can view this awkwardness is that it's a necessity since Postgres requires the conflict target. So there is a few things within this code.
1. In order to get an id back when there's a conflict,we must make the statement an update and therefore we need to define a strategy a proper strategy (`[set: [name: name]]`), `:nothing` will not do.
2. You cannot upsert without specifying the `conflict_target`. The compiler will yell at you if you only have the `on_conflict` with the **strategy** defined.

## Why does insert_all go back to `on_conflict: :nothing` and no `conflict_target`
Now that hopefully you're convinced that the two things you **need** to do to allow for upsert is to 
1. Use a `on_conflict` strategy
2. Add the `conflict_target`

I am here to tell you that you are wrong! Well it's only partially true. You need those two options when defining upsert with `insert/2`. Upserts
with `insert_all/3` is different and looking at the logs reveals why. There are two queries made when using upsert with `insert_all` whereas an upsert with `insert` uses a single one.
```bash
19:01:16.597 [debug] QUERY OK source="tags" db=9.0ms decode=0.7ms queue=10.3ms idle=1731.2ms
INSERT INTO "tags" ("inserted_at","name","updated_at") VALUES ($1,$2,$1),($1,$3,$1) 
ON CONFLICT DO NOTHING [~N[2024-03-08 19:01:16], "elixir", "ecto"]
 
# a read statement that was not there before
19:01:16.626 [debug] QUERY OK source="tags" db=21.0ms queue=0.3ms idle=1763.8ms
SELECT t0."id", t0."name", t0."inserted_at", t0."updated_at" FROM "tags" AS t0 
WHERE (t0."name" = ANY($1)) [["elixir", "ecto"]]
```

The guide references this as well two queries however I wasn't aware of this. The two queries are:
1. Write: inserts all the items into the table with a single query
2. Read: retrieves all the entities we've inserted

Therefore by declaring `on_conflict: :nothing`, we're able to ensure that we successfully insert all the tags with no errors. The second query
that retrieves them all will properly have their ids which allow us to properly associate the post with the tags.

The reason we have an additional query is due to the slightly different ways that we return data after the insert. Recall that when inserting the 
single insert the SQL statement actually specifies that it should return something! Inserting all does not return anything, in this case I want to 
say that it's because it's too difficult to do so in one SQL line. Therefore we must have the second query to retrieve all the things we've inserted.
```bash
INSERT INTO "tags" ("name","inserted_at","updated_at") VALUES ($1,$2,$3) 
ON CONFLICT DO NOTHING RETURNING "id" ["newlo", ~N[2024-03-08 16:42:41], ~N[2024-03-08 16:42:41]]
```

## Putting it all together
### Upsert with insert/2
The SQL queries formed by Ecto will actually add the `RETURNING ...` to the insert queries. We are told that for upserts to work we need `on_conflict` to be defined. 
Using `:nothing` causes the insert to not complain but since this insert fails it will not return an id, causing our insert of a Post to fail since 
its associated entity (a tag) has an invalid id with the value of `nil`. This forces us to 
1. Define the `conflict_target` so that we can define the rules of when the insert should turn into an update
2. Define a strategy for `on_conflict` which defines the rules of how the update should change the row

### Upsert with insert_all/3
`insert_all`, `update_all`, and `delete_all` all behave similarly in that the result in two queries. 
1. To change all the data
2. To retrieve all the data

Since we're able to retrieve all the rows with their ids, we don't run into the same problem that we did with `insert` which caused us to define both `on_conflict` and `conflict_target`.
Therefore **you only at least need `on_conflict`** for an upsert.



