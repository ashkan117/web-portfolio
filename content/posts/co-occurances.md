---
title: "Co Occurances"
date: 2025-02-01T11:57:28Z
draft: true
---

## All things pairs

The goal of the problem was to create a function that given two words would
return the co-occurence of those words. Where the definition of
co-occurence is the number of lines in which two words appear together

I think I got mixed up on a few things with the wording of the problem,
I should have

1. Had a pen/paper to write down the definition of the term.
   I assumed that it would be defined on the screen for me.

I found essentially the same exact problem discussed on [leetcode](https://leetcode.com/discuss/interview-question/640961)

## The Goal of the Problem

The restrictions of the problem was to solve this problem using two functions

```python
def update(line: str):
    pass


def get(a: str, b: str) -> int:
    pass

```

What I did not understand was that we didn't care about the time complexity of `update`,
we just wanted the best time complexity for `get`

## How to properly iterate through all the pairs

This was where I could have had some saving grace but I messed this up.

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
  for y in range(x):
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
  for y in range(x + 1):
```

## Reading the code

## All My Work

To conclude, I will include the full file that I worked on

```python
from collections import defaultdict
# alice bob clair
# bob clair dan
# junk junk
# alice bob clair

# the co-occurance is the number of lines in which two words appear together

# c(alice, bob) == 1
# c(alice, dan) == 0

memory = defaultdict(int)


def update(line: str):
    words = line.split(" ")
    for word_one in words:
        for word_two in words:
            key = sorted([word_one, word_two])
            if tuple([word_one, word_two]) in memory:
                memory[tuple([word_one, word_two])] += 1
            else:
                memory[tuple([word_one, word_two])] = 1


def get(a: str, b: str) -> int:
    print(memory)
    return memory[tuple([a, b])]


def main():
    line_one = "alice bob clair"
    line_two = "bob clair dan"
    line_three = "junk junk"
    line_four = "alice bob clair"
    update(line_one)
    update(line_two)
    update(line_three)
    update(line_four)
    assert get("alice", "bob") == 2
    assert get("clair", "bob") == 3


if __name__ == "__main__":
    main()
```
