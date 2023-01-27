+++ 
draft = false
date = 2023-01-27T11:16:59-08:00
title = "Quarrel With Dynamic Programming"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

# DP is an Art

# Snapshot is clear, global is not
An issue I ran into with developing an intuition with dp is that in some cases the immediete solution is obvious yet I had a nagging sense of how could it be right long term. To explain my issue I'll go through a leetcode problem. https://leetcode.com/problems/min-cost-climbing-stairs/description/

The case specifially which led to this is one where we can see a large cost being unattractive short term but my intuition was telling me couldn't it be possible that in the long run this is good?

Input: cost = [1,100,1,1,1,100,1,1,100,1]\
Output: 6\
We can see that the 100s seem like mines that we want to avoid at all cost. However in my head there seemed like a possibility that when we're taking the last step stepping on one mine is fine if that is the only mine. What if future paths ended up having way more mines on its path to the last step?
Another way to explain why things are a little unclear are by imagining the recursive solution.\
cost[i] + min(recursiveCall(cost, i - 1), recursiveCall(cost, i - 2))\
In the recursive view of things my issue is that the recursive calls have no context of the mines that are along the way so couldn't one of the recurve calls contain a mine that seems bad in the bottom up approach

In order to show this then I need to find a way where we can show that there is a way to reform the cost array to have a mine early on that ends up being an optimal path.
Input: cost = [1,100,1,1,1,100,1,1,100,1]\

Let's change it to where maybe the 100 seems bad now but there are way worse options
Input: cost = [1, 100, 2000, 2000, 2000, 2000, 2000, 2000, 2000, 1], length = 10\
The 100 looks pretty good now. Let's change the question so you can jump 8 steps. So the optimal would be 100 + 1, since 1 + 2000 is worse
```
currentCost + min(    recursiveCall(step1) , ... , recursiveCall(step8)    )
```
**They key is that once you make the decision it's optimal. The future step depends on these optimal subproblems. The only way to get here is through its predecessor.**

# Optimal Substructure
https://cs.stackexchange.com/a/79749

A characteristic of dp problems are that they contain an optimal substructure. This means that an optimal solution must depend on an optimal solution from a subproblem
from wiki
> Typically, a greedy algorithm is used to solve a problem with optimal substructure if it can be proven by induction that this is optimal at each step.[1] Otherwise, provided the problem exhibits overlapping subproblems as well, 
> divide-and-conquer methods or dynamic programming may be used. If there are no appropriate greedy algorithms and the problem fails to exhibit overlapping subproblems, 
> often a lengthy but straightforward search of the solution space is the best alternative.

# Proving Optimal Substructure
Going back to the min climbing stairs problem lets prove it. Let's say we have a suboptimal solution to a subproblem that we can choose to try and build our optimal subproblem. We'll do an intuitive proof using proof by contridition
Input: cost = [1,100,1]\
Let's say we're at index 2

We see that if we took two steps before this the cost is 1, if we take one step the cost is 100. We see that by picking the suboptimal subproblem we end up with a suboptimal solution. 


