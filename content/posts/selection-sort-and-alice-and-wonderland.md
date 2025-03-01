---
title: "Selection Sort and Alice and Wonderland"
date: 2025-02-11T20:06:43Z
draft: true
---

## Selection sort and tennis matches

In reviewing some of the basic sorts I always find myself forgetting which one
is which for the most part. Bubble sort is easier to visualize, and insertion
makes me think of poker but the rest not
so much. In an attempt to properly remember selection sort I wanted to take
a look into its history. It is way more interesting than you think. My
goal here is to show you how the author of Alice and Wonderland is accredited
with formalizing the general sorting algorithms called **selection algorithms**
but also that he did this through tennis matches.

## Why is it called selection sort?

Let's hone in on what the selection sort is. Then we'll take a look why Lewis
is accredited with the family of algorithms tied to it, **selection algorithms**.

> “The simplest method for sorting is to repeatedly select the smallest
> remaining element and move it to its final position. This process leads to
> what is known as selection sort.”\
> — Donald E. Knuth, The Art of Computer
> Programming, Vol. 3: Sorting and Searching (1973), Section 5.2.3

I think at its crux the algorithm really is

- Picking some selection criteria that is used each round (select the minimum)
- Each round we shrink the solution space
- Each round we go through all possible items in the solution space

I think this animation from [wikimedia](https://upload.wikimedia.org/wikipedia/commons/3/3e/Sorting_selection_sort_anim.gif)
serves as a nice visual for this intuition.

![wikimedia](https://upload.wikimedia.org/wikipedia/commons/3/3e/Sorting_selection_sort_anim.gif)

## A "fair" tournament

Now that we can visualize the selection sort algorithm a little bit better
let's take a look at Lewis' tennis matches. More specifically lets take
a look at what is we selecting each round and what the end result is.

His motivation was to end up with "fair" results of a tournament.
This means that assuming you can accurately rank the players, then
you end up with a final result where the winners will essentially
sorted by rank.

## Resource

- [Lawn Tennis Tournaments (1883) by Lewis Carroll](https://schnark.github.io/lewis-carroll/html/texts/lawn-tennis-tournaments-1883.html#5-an-equitable-system-for-scoring-in-matches)
- [Donald Knuth](<https://seriouscomputerist.atariverse.com/media/pdf/book/Art%20of%20Computer%20Programming%20-%20Volume%203%20(Sorting%20&%20Searching).pdf>)
- [Accrediting Lewis](https://people.csail.mit.edu/rivest/pubs/BFPRT73.pdf)
