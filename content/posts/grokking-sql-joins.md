---
title: "Grokking Sql Joins"
date: 2024-01-25T10:05:43Z
draft: true
---

## Introduction
 
Joins are a fundamental part of SQL yet it always felt like something didn't click for me. When writing complex SQL queries it
usually felt more like following a formula that you don't truly understand rather. While working on more complex problems it requires
more complex queries. This post serves to create a conceptual model of how to view forms at a more intuitive level.

## Venn Diagrams

So the best way to learn things is visually. If you l 
Joke


## Normalized Data

In SQL, the way we organize data is through normalizing the data. Intuitively this means that whatever entities you have are broken into 
chunks. SQL queries with joins are really combining that collective data back into a whole. Therefore the reason we need joins are a side effect
of normalizing our data. In a No-SQL database you could just store everything in a single location called a document. There's no need to patch together your entity 
in No SQL databases since the logical organization of the data is different.

## Foreign Keys are Connection Points
Let's say you have a Todo list application. If you organize your todos as a project then when you want to load data about your project 

## Outer Joins (Left Join and Right Join)

Outer joins are what made me motivated to actually start this post. I would have some challenges with left joins and run into queries returning
duplicate data without really understanding why.


### Optional Data

Many entities have data related to them that are not guaranteed to exist. If you're storing information about tenants in an Apartment complex,
then pets are optional. People could have pets or they could not. With the todo example, people could have created a Project with no todos. Some data
inherently has is not fixed and predictable. 


### Do you care about the optional data.

Let's say you're looking at your messages on your phone. Now let's imagine you were going to build the query to get back the messages.
Our database in this example will have three entities `User`, `Message`, and `Attachment`. A message will have a user but it *could* have 
an attachment. Let's build a query that will properly get all the messages we've received. Let's start with

```sql
SELECT * FROM messages m
INNER JOIN users ON users.id = m.receiver_user_id
```

This gets you back all the messages but what about attachments? How do we join them?
Do we care whether or not a message has an attachment for this query? No we don't. If we want to 
display all the received messages we still want to return messages with no attachments! This means
we really want an outer join. This is the same as saying still return the data if there is no 
concrete match. 

```sql
SELECT * FROM messages m
INNER JOIN users ON users.id = m.receiver_user_id
LEFT JOIN attachments ON attachments.id = m.attachment_id
```
Now what if we have a checkbox that allows us to filter on only the messages with attachments? Now we 
do care if there are attachments and we only want the messages with a valid "connection" to be returned.
The context of the question is now very different. It still returns the same entity, contact in this case, 
yet the way we connect the entities together are subtly different. 

## Entities as trees

You can think of an entity as a tree where each entity FK relationship represents a node. In our earlier example we can then
view messages as 3 nodes pieces. The root one to represent the messages and two children nodes that represent the receiver_user_id
relationship and one for the attachment_id.

If the FK relationship exists, we can think of that node as filled in. Otherwise it's hollow inside. A message will always 
have itself. In this case a message will always have a user. We can think of this one as a requirement we chose for our application.
For the root node and the node that represents the user, these will always be filled in. 
The more interesting case is the attachments. Since it makes sense for messages not to always have attachments, this node is sometimes
filled sometimes hollow.

Therefore if we looked at all the messages, we'd always get back two forms. One's with everything filled in (messages with attachments)
and one's where the attachment node is hollow (messages with no attachment). Therefore when making a query we should be mindful in which 
ones do we want returned. Do we want both kinds? Or maybe we just want the ones where they are all filled in? **If you don't care if they are filled in, that is when you want an outer join.**

In the first example where we wanted all messages we didn't care if they had an attachment, therefore we wanted both trees that are all filled and hollowed.
In the second situation where we wanted to filter messages to only return the ones that have an attachment, this would mean we only want the messages 
that have all the nodes filled.
