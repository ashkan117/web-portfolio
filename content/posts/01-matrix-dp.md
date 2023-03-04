+++ 
draft = false
date = 2023-02-12T17:02:24-08:00
title = "01 Matrix (Solving DP by splitting it into pieces)"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

# Trouble
Even if you traverse the node wouldn't you need to traverse the node for every node.
For each node we'd need to determine the distance by traversing it?
Multiple traversals are costly

A Dynamic Programming solution comes into play. Ask your neighbor what is the distance to 0 from you. Pick the neighbor with min distance

# DP solution

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
