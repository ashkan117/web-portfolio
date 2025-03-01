---
title: "Iterating through all pairs"
date: 2025-01-29T14:27:24Z
draft: true
---

## All things pairs

I recently had an interview an interview where the solution
required me to iterate through all the pairs of words in a sentence.

I was a little rusty and created the nested loops in the wrong way.
I was relying on memory rather than understanding. I wanted to strengthen
my intuition on the code and that's what this post is about.

Let's start with my mistake first.

```python
# my way, the wrong way
for word_one in words:
  for word_two in words[1:]:

# the right way
for i in range(len(words)):
  for j in range(i + 1, len(words)):
```

I remembered there was a one there but I mixed this up because in an interview I
trusted my memorization rather than understanding things.

## What are all the possible pairs to begin with

Well lets say we want to find all ways we can pair numbers from [0,3]. I think the
simplest way to think of this is to fix one number for the x pair (start from
the smallest number), then to move the other number. So let's say x = 0,
then we'll start y.

```bash
(0,0) -> (0,1) -> (0,2) -> (0,3)
```

Now we can increment the x by one and repeat the same process.

```bash
(1,0) -> (1,1) -> (1,2) -> (1,3)
```

Nice, let's do that for the remaining and we're left with the following

```bash
(0, 0) (0, 1) (0, 2) (0, 3)
(1, 0) (1, 1) (1, 2) (1, 3)
(2, 0) (2, 1) (2, 2) (2, 3)
(3, 0) (3, 1) (3, 2) (3, 3)
```

The code for this is not too bad. Let's look at a few details

- We start x at 0 and y is incrementing
- y starts at 0 every new row

```python
for x in range(3):
  for y in range(3):
    pass
```

## Unique Pairings

There's one last thing we need to look at that tends to matter in some problems.
Are the pairs unique? This completely depends on context of the question.
Let's say (x,y) represents a pair of people. If we were running some psychology
experiment relating to couples, maybe we'd want to define things mathematically
and say that inRelationship(John, Jane) represents if a couple is in a relationship.

In this case inRelationship(John, Jane) is the same as inRelationship(Jane, John).
THere's a symmetry to it since if John is in a relationship with Jane (and he's not
some crazy stalker) then Jane is also in a relationship with John. This is the
crux of unique pairing, due to this symmetry we don't actually want to
calculate all possible pairs of (x,y).

## To Repeat or not to Repeat

One other consideration is repetition. All that means is
(x,x) valid? Again this is based off the context. Going back to the psychology
example we'd could actually argue that (x,x) is valid. Not sure about the legality
but some celebrities will be in the headlines for "marrying" themselves. So
inRelationship(Brittany Spears, Brittany Spears) could be argued true. But
again, this is based off context so it's completely reasonable that in our
experiment we don't want to consider that relationship.

### Unique Pairings WITH Repetition

Visually these pairing will look something like the ones below.

```bash
(0, 0) (0, 1) (0, 2) (0, 3)
       (1, 1) (1, 2) (1, 3)
              (2, 2) (2, 3)
                     (3, 3)
```

The reason there are missing pairs is again that unique pair concept. If (0,1)
is there we don't need (1,0) when we're only looking for unique pairs.

How can we code this? We'll we see a few similarities from before. At each
iteration x stays fixed, y is incremented.
However the main difference is y never starts at 0 anymore,
it always starts at whatever x is on the current row.
When we start with x = 0, y is also equal to 0. On the next
row we increment x to 1 and instead of starting y at 0,
we start y at 1. The starting point for y
is always the same as the starting point for x.

```python
for x in range(3):
  for y in range(x, 3):
```

### Unique Pairings WITHOUT Repetition

Finally the last possibility for unique pairs is if we don't allow for repetition,
meaning (1,1), (2,2), and (3,3) are not valid pairs. This leaves us
with the following pairs.

```bash
       (0, 1) (0, 2) (0, 3)
               (1, 2) (1, 3)
                      (2, 3)
```

```python
for x in range(3):
  for y in range(x + 1, 3):
```

## Understanding the Triangle

I think the first thing required to really understand the code is that we want to
form this upper triangle. All pairs look like the following

```bash
(0, 0) (0, 1) (0, 2) (0, 3)
(1, 0) (1, 1) (1, 2) (1, 3)
(2, 0) (2, 1) (2, 2) (2, 3)
(3, 0) (3, 1) (3, 2) (3, 3)
```

If we only care about the unique pairs then we'd need to eliminate
the non unique pairs. There's a symmetry for (x,y) and (y,x). Along
the middle of the grid, if we look across we'll see the complement pair.
The middle in this case is (0,0), (1,1), (2,2), and (3,3).
So if we look at the diagonal of (2,2) we can see (1,3) and (3,1).

```bash
# if we eliminate the non unique pairs in the upper triangle
(0, 0)
(1, 0) (1, 1)
(2, 0) (2, 1) (2, 2)
(3, 0) (3, 1) (3, 2) (3, 3)

# if we eliminate the non unique pairs in the lower triangle
(0, 0) (0, 1) (0, 2) (0, 3)
       (1, 1) (1, 2) (1, 3)
              (2, 2) (2, 3)
                     (3, 3)
```

## Deriving the Upper Triangle

### Fix x and increment y

We can think of it in words. I want x to stay fixed and y to increment.

```python
x = 0
for y in range(0, N):
  # will print (0,0) (0,1) (0,2) (0,3)
  print(x,y)
```

### Repeat this again for all possible values of x

We want this work to repeat for all possible values of x.
After I get all pairs with x, then I want to increment x so that I can
get all pairs for x + 1.

```python
# we want x to start at 0 and increment till the end
for x in range(0,N):
  for y in range(..., N):
    # will print (0,0) (0,1) (0,2) (0,3)
    print(x,y)
```

### Eliminate the non unique pairs

We know we need a loop but again our goal is to have unique pairs.
Remembering the structure of the triangle we know that it's a triangle.
We don't have any elements in the lower left side since we already
found the needed pair in the upper right triangle.

We therefore remember that the bottom of the triangle is
really the identity pairs line. (0,0) (1,1) (2,2) (3,3).
This helps us realize that the y value will always start at x.

```python
for x in range(0,N):
  for y in range(x, N):
    print(x,y)
```

### Maybe remove the identity pairs

Final decision is do we allow the repetition of (x,x)? If not we just make sure that
y starts at x + 1 instead.

```python
for x in range(0,N):
  for y in range(x + 1, N):
    print(x,y)
```

## TLDR

1. Keep x fixed and increment y
2. Repeat this work for all x values
3. Eliminate the non unique pairs
4. Maybe remove the identity pairs
