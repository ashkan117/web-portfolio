---
title: "Coin Change"
date: 2023-04-11T16:24:25-07:00
draft: true
---

## Intuition
https://labuladong.gitbook.io/algo-en/iii.-algorithmic-thinking/detailsaboutbacktracking#permutation

## The Problem
You are given an integer array coins representing coins of different denominations and an integer amount representing a total amount of money.
Return the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return -1.
**You may assume that you have an infinite number of each kind of coin.**

The way the recursion tree is built is that it would try the first element as much as it can. To understand things better let's take a look at the 
example when we have coins = [1,2,5] and amount = 6.


## Understanding our decision tree
0. Nodes: Understanding the subproblem
1. Path: the selection that you want to make.
2. Candidates: the selection you can currently make.
3. End Condition: the condition under which you reach the bottom of the decision tree, and can no longer make a selection.

## Nodes: Understanding the subproblem
We have an amount and we have several options to choose from in this case let's not care and just choose the first one.

## Building our Path / Making our Selection


## How can we code the naive solution
The way the recursion tree is built is that it would try the first element as much as it can. To understand things better let's take a look at the 
example when we have coins = [1,2,5] and amount = 6.

Coding the recursive solution to any question requires you to understand how to solve it through wishful thinking. That is, what is the laziest way you can solve this problem. 
If I wanted to be really lazy I would say I want to use the coin with value 1 to solve this problem when amount is 6. If I do that these is only a valid answer if a solution
for coinChange(5) exists.

We'll continuously pick the first element as much as we can and recursively try and solve that subproblem.
```
coinChange(6)
  coinChange(5)
    coinChange(4)
      coinChange(3)
        coinChange(2)
          coinChange(1)
            coinChange(0) -> returns 0
            coinChange(-1) -> returns -1
            coinChange(-4) -> returns -1
          -> returns 1
```

### Our base case
How do we know when we've found a solution? In this case we can think of it as when we don't overshoot. If the amount is 2 but the only coin we have is 3 then there is no valid solution since if we pick 3 our amount would be -1.
If we have a coin with value 2 though that would work. Essentially this makes our base cases the following.
```python
if rem < 0:
    return -1
if rem == 0:
    return 0
```

It will continuously try to solve the subproblem while picking a coin with the value of 1. Eventually we'll reach the subproblem where we have amount = 1. In this case we can see that the next time we call it
we'll end up with our base case of amount 0 which means we've found a valid solution.

## How the recursion unwinds?
Now that we've found the minimum solution from coinChange(1) we can return to the subproblem that called us coinChange(2). Remember that the recursive way of thinking in this problem is
if I know that there is a solution for the complament of amount - coin_interested_in then a valid solution exists. 
When we wake up in coinChange(2) it is when we just finished using the first element as much as we could. In this case it returned since we found the minimum solution for coinChange(1)
Now we need to determine are there any other coins that can solve this problem better? So we will try the next available coin which is 2.
```
coinChange(2)
    coinChange(1) -> returns 1
    coinChange(0) -> returns 0
    coinChange(-3) -> returns -1
```

In coinChange(2)'s context this means that since coinChange(1) used 1 coin 

### Optimal Subproblem
**Notice that after we've tried all three coin options the minimum we receive from there is the optimal minimum.**


### Bonus: There's no point in continuing if we've found a solution?
In the case where we only picked 1 and reached a solution there is no way that any other coin could also be a solution. After we pick 6 ones we back track to the case where we had an amount of 1 left. 
Since we know that we've already found a valid answer it doesn't make sense to pick coins 2 and 5 since they would overshoot.

However on coinChange(2) the first solution we find is when we have all ones so the minimum coins used is 2. Does this mean we should not try and use another coin? No, you can see that when the amount is 2 then we can just use a single 2 coin and our
minimum coins becomes 1.
```
coinChange(6)
  coinChange(5)
    coinChange(4)
      coinChange(3)
        coinChange(2)
          coinChange(1)
            coinChange(0) -> returns 0
            coinChange(-1) -> returns -1
            coinChange(-4) -> returns -1
          -> returns 1
```

```python
min_cost = float('inf')
for i, coin in enumerate(coins):
    res = dfs(rem - coin)
    if res != -1:
        min_cost = min(min_cost, res + 1)
return min_cost if min_cost != float('inf') else -1
```
