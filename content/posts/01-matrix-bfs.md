+++ 
draft = false
date = 2023-03-01T11:23:30-08:00
title = "01 Matrix (BFS with multiple sources)"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

# Introduction
We're taking a look at [01 Matrix](https://leetcode.com/problems/01-matrix/description/) on Leetcode today. The main motivation is to realize that when using BFS you can start with multiple sources instead of a single source. This just means that even though we usually start out with a "root" and do BFS from there, you can also start from multiple "sources" and the you get the same benefits of BFS. So we can use this form of BFS to find the minimum path from all 1s to 0s.

There is another solution related to dynamic programming if you're interested in that as well.

# Why BFS?
Once you see it as a BFS problem it's hard not to see it later. However the main reasons are:
- There's the idea of path (Finding a path from 1 -> 0)
- We're looking for minimum path's which is a huge motivation behind BFS

# Trouble with thinking about a solution as a path from 1 -> 0
The question asks for "return the distance of the nearest 0 for each cell" which makes it natural to start thinking of finding the path starting from 1. You can find a solution with that approach however you can find a much more
effiecient approach by flipping it to "find path from 0 -> 1". 

I just want to take sometime to talk about the fact that the efficiency changes based off that view which is a little crazy at first since they seem like mirrored solutions. **The main difference is one takes
advantage of BFS's properties more than the other** leading towards a better time complexity. Let's see how starting with looking at the cell in the middle at (1,1).

![image](/images/01-matrix.jpg)

Let's say we figure out the path of this cell using BFS. We use BFS to search for the closest 0 to this cell. This allows us to confidently say we've found the closest 0 to this 1. 
Now let's look at cell in the bottom left corner (2,0). Assuming we know the result from the previous BFS can we use that in some way? Not really since these 1's are not connected in any way. 
How much does flipping the approach from path from 1 -> 0 to 0 -> 1 actually help us?

Now let's say we're starting from 0 and finding a path 1 to. Let's start at the 0 found in (1,0). With a single iteration of BFS from this 0 source and we have actually found **all minimum paths to a 0 from ALL neighboring 1s**. That is the major difference! If we start from 1 going to 0 we need to perform multiple BFS searches since we only find a single solution at a time. In our earlier case by performing a full BFS from (1,1) we've only discovered the minimium path to a 0 for that single cell. To find the path for all other
1s we need to redo the entire BFS search! By flipping the approach we find all possible solutions by doing a single BFS starting at multiple sources (all the 0s).

# Intuition
## Which traversals to use
Breadth First Search with Multiple Sources

We're looking for a minimum distance so this should be a cue to use BFS
## Flipping it to start from 0s
One way you can look at solving this problem is starting at the 1s then search for a nearby 0. The reason this isn't the easiest way forward is because there is an air of uncertainty. Let's say we start at a one and that one is surrounded by all 1s. Our search is not done so we'd need to continue the search until we find a 0. Using BFS you can guarantee that this would be the minimum distance to a zero however we're doing all this work to find the nearest 0 of single node.


A simple change of starting from node 0 and looking for nodes' with value 1 has a few added benefits.
1. You can find the minimum distance of more nodes faster

The issue with the first approach is that you'd need to do BFS for all nodes with the value of 1. By switching it to start with the 0 nodes you would only need to do it once.

0 0 0\
1 1 0\
1 1 1

If we start from the one at index (2,1) then we do BST we can find the minimum path to 0 for that specific node. There is a way to make that information that we gained relevant to other node's but for now notice that the work we just did found a single minimum path. If we reverse it ot start from the 0 (the one at index (0,1) we just found notice we actually solve the problem for multiple nodes.

After one iteration of the BFS we found that there is a one below us. This guarantees that we've found the minimum distance from a 0 to that 1. We've already found the minimum path for one node.The 1 node at (1,1) has a 0 node 1 away. Next iteration we add two more one nodes to our queue. We've found the minimum distance to these nodes as well since it's the 2nd level of our BFS. We know that these 1 nodes are a distance 2 away from a 0.

Hopefully, this example serves as a good intuition why starting from 1 -> 0 is not as useful as 0 -> 1.

## Can we have multiple sources

We can see that BFS makes a lot of sense when we have a single source, meaning that if there's a single starting point. In our case if there is a single 0 things are easy. Just do BFS starting from that node. We know that
we can use this to find the minimum distance from the 0 to the 1. Does this change if we have multiple sources? If we have multiple 0s we could start with how do we know which 0 is the one that's closer to some 0?

0 1 1\
1 1 1\
1 1 0

In the two 0 example you can see that there are some 1s that are closer to one 0 than the other. How would BFS handle this? It handles it perfectly.

queue = [root]
queue = [source1, source2]
This works because we start source1 then we add all the neighbors one step away. If any of these nodes one step away have the value 1 then we've found the minimum distance.
This is since the minimum distance from a 1 node to a 0 node is one. We popleft and now we're looking at source2. We do the same thing. Look at all the neighbors.
If any of the neighbors has the value 1 then we know that this is a minumum distance from 0 -> 1.

### Why does this work.
If we only take one step at a time then the first source that reachees a particular node must be the shortest path to that node.
Lets say we have two sources
source1 and source2
source1 goes to all its neighbors one distance away, adds them to the queue
source2 goes to all its neighbors one distance, adds them to the queue.
now that we're looking at all the neighbors of source1 with a distance of 1 away and look at all its neighbors with a distance of 1 away. We're looking at the nodes with a distance 2 from source1.
We know that all these sistances are the minimum path. We're saying that these neighbors with a distance of 2 are closer or equal to source1 rather than source2.
We can show this with a Proof by Contradiction.
If source2 is closer then it would have a distance of 1. That can't be true since if we had a distance of 1 then it must have already been added in the first step.
