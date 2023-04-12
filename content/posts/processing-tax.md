---
title: "Processing Tax (Graph Traversal)"
date: 2023-04-07T12:32:23-07:00
draft: false
---

## Goal
The goal of this blog is to discuss graph traversal algorithms and give a little more intuition between the flow of the actual code. The blog assumes you are already familiar with Breadth First Search (BFS) and Depth First Search (DFS) algorithms. I would always 
forget when do you add a node to the visited set or when do you validate adding node to the stack. This hopefully makes things stick better.

## Introduction
Many problems can be structures in terms of graphs which makes graph traversals very powerful tools. Think of all the billion dollar companies that started with that simple approach. Some examples are:
- Friend networks (Facebook, Instagram, Twitter, LinkedIn)
- Maps Networks (Google Maps, Waze, Apple Maps)

The world is all about relationships and graphs are one of the languages needed for that.

## A History of our Path
When we look through some algorithms that require graph traversal there is a requirement that we have some way to detect whether we have seen this node before. That's because almost always you only want to "process" a node once. 
The idea of processing a node depends on the problem. One example could be given a graph you want to calculate the total value. In a [https://leetcode.com/problems/course-schedule](scheduling problem) you might be interested to see how this current
node relates to our current interpretation of the "correct" schedule. 

Regardless the key here is that we only want to process a node once so how can we make sure we do that? Let's make things a little more visual.

> Everybody knows that all you need to explore a labyrinth is a ball of string and a piece of chalk. The chalk
> prevents looping, by marking the junctions you have already visited. The string
> always takes you back to the starting place, enabling you to return to passages that
> you previously saw but did not yet investigate.

We're focusing on how to simulate the chalk section of that intuition. This is actully pretty easy since we can use the concept of a set. A list that is guaranteed to have unique number. So when you walk up to a fork in the road in the maze we 
can take a look at this list and see which way have we already visited so that we can go the unvisited route. The thing that didn't click for me for a long time was when do mark something as visited in code?

## Broken Code
I'm assuming you've already at some point coded DFS before. 
I'm using a the code with a stack since there's a subtle detail that I missed during a programming interview question that 
The general code focusing on the traversal side is
was similar to [https://leetcode.com/problems/number-of-islands/](number of islands). 

```python
def dfs(r: int, c) -> Tuple[int, int]:
    stack = [(r,c)]
    while stack:
        r, c = stack.pop()
        visited.add((row_offset, col_offset))

        # for child in children
        dir = [(1,0), (0,1), (-1, 0), (0,-1)]
        for x, y in dir:
            row_offset = r + x
            col_offset = c + y
            if (row_offset, col_offset) in visited:
                continue
            # if neighbor we're considering is actually valid
            if 0 <= row_offset < N and 0 <= col_offset <:
                stack.append((row_offset, col_offset))
```

# The problem 
My bug was that I marked the node as visited at the wrong place. The assumption I made is that we consider a node as visited 
as soon as you "process" it (popping it from the stack).

The reason why that is wrong is this leads to a situation where we ended up processing things multiple times. This made no sense to me!
If we're specifically doing DFS then it must go as far as it can, there is no way we hit the same node twice while going down 
one path. Even more genrerally graph traversals don't really visit a node twice. 

The problem occured when we had a loop in our graph. Think of the example below and we're following the path of 1s. 

```python
  ["0","0","0","0","0"],
  ["0","1","1","1","0"],
  ["0","1","0","1","0"],
  ["0","1","1","1","0"]
```

I start at (1,1) then traverse things all the way until I reach (1,1) again. If I never added (1,1) to our visited set then 
when we're at the last node before (1,1) which is (2,1) then we would add (1,1) to the stack again.
Then at some point we would have [(1,1), (1,1)] in our stack. 

Everything that is in our stack must be already in the visited set since there could be things in our stack 
that we have not "processed" but they are in our stack. These elements must already be considered or else our
algorithm won't know better and attempt to visit them.

> The "process tax" is saying that you must make sure that our visited set knows about the node we're about to explore. Before you
fully process something you must pay the tax of adding it to the visited

## Conclusion
The visited set is linked with the stack. Once you **plan** to do work is when you add things to the vistited set. In the 
maze analogy this means you mark down things as visited right before you trek off into the unknown or you can also do it as soon as you start the path. You just turn around mark it as visited then continue.

## Working Code
```python
def dfs(r: int, c) -> Tuple[int, int]:
    stack = [(r,c)]
    visited.add((r,c))

    while stack:
        r, c = stack.pop()
        visited.add((row_offset, col_offset))

        # for child in children
        dir = [(1,0), (0,1), (-1, 0), (0,-1)]
        for x, y in dir:
            row_offset = r + x
            col_offset = c + y
            if (row_offset, col_offset) in visited:
                continue
            # if neighbor we're considering is actually valid
            if 0 <= row_offset < N and 0 <= col_offset < N:
                stack.append((row_offset, col_offset))
                visited.add((row_offset, col_offset))
```


