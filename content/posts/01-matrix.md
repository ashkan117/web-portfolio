+++ 
draft = false
date = 2023-02-12T17:02:24-08:00
title = "01 Matrix (BFS with multiple sources, solving DP by splitting it into pieces)"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

:ROAM_ALIASES: "Leetcode 542"

# Trouble
Even if you traverse the node wouldn't you need to traverse the node for every node.
For each node we'd need to determine the distance by traversing it?
Multiple traversals are costly

A Dynamic Programming solution comes into play. Ask your neighbor what is the distance to 0 from you. Pick the neighbor with min distance

# BFS
What do we do if an answer is not ready? Like what if everything else is 1, don't we need to do DFS?
Only BFS

Do BFS on every node that is 0. If there is just one node its actually really simple and you can do this in O(n) time.
However it's trickier if there are multiple 0s since there could nodes where one zero is closer than the other

1 1 0\
1 1 1\
1 1 1

1 1 0\
1 1 1\
0 1 1

This one makes it seem more like DFS again. A little


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




## DP solution

Two direction approach
We know that we could do the minimum of all 4 directions plus 1. The issue that happens is that how do we guarantee that we've already calculated that distance

```python
dp[1][2] = 1 + min(
                    dp[0][2] the cell upward
                    dp[2][2] the cell below
                    dp[1][1] the cell left
                    dp[1][3] the cell right
                  )
```

we can do this by controlling the directions
If we start from 0,0 then only do the left and down direction we're guaranteed that the previous ones were done
A detail is that this iteration assumes that the leftmost topmost cell is the minumum value.

0 0 0\
0 1 0\
1 1 0

If the element is 0 we don't do any work which makes our dp structure

0 0 0\
0 X 0\
X X 0

We're iterating through the matrix starting at 0,0 then nested loop
0,0 0,1 0,2\
1,0 1,1 1,2\
2,0 2,1 2,0

So 1,1 depends on left and top which are already available since we're guaranteed to have done 0,1 and 1,0

```python
dp[1][1] = 1 + min(dp[0][1], dp[1][0])
         = 1 + min(0,0)
         = 1
```
0 0 0\
0 1 0\
X X 0

**So just look at your cell, determine your value based off who is on top and who is to your left. if No element exists assume infinity**

0 0 0
0 1 0
1 2 0

Now let's do the other two directions
Assume matrix[len(rows)][len(cols)] is minimum and go from there
So now we're building off the cell below us and to the right

Note we're iterating through from reverse so we goes
If we're doing a nested decrementing loop we visit the pairs in this specific order.
We only pick two specific directions at a time since that's the way we iterate through the array easily.
2,2 2,1 2,0
1,2 1,1 1,0
0,2 0,1 0,0

**So we're looking below us and to the right at each cell. If no element exists then assume infinity.**

0 0 0
0 1 0
1 2 0

At (2,2) there's nothing below us or to the right so leave alone.

```python
dp[2][1] = 1 + min(dp[2][2], dp[3][1])
         = 1 + min(0,infinity)
         = 1 + 0
         = 1
```

0 0 0\
0 1 0\
1 1 0

Notice that the dp[2][1] self corrected. Now we correctly considered all four directions and picked the minumum of those. Now after you place a value in dp[i][j] that is the optimal solution.
So we're revisiting each cell making sure that we made the right choice now that we're including all directions

0' 0' 0'\
0' 1' 0'\
1' 2' 0

You can assume that this two pass system means that the first pass is marked with a ' to specify that we're not done yet
0' 0' 0'\
0' 1' 0'\
1' 2' 0

After you do the second pass on the cell then we're done and could remove the mark

0' 0' 0'\
0' 1' 0'\
1' 1  0

**We only do it in two passes since it makes iteration doable.**
