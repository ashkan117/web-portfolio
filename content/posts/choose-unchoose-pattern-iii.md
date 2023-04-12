---
title: "Choose Unchoose Pattern Iii"
date: 2023-04-11T18:49:59-07:00
draft: true
---

## Recognizing the Pattern
In this part we'll just focus on how to better recognize this pattern as well as variations in the code that show up.

## Regular Power Set 
Let's say the set is:

A = {1, 2, 5}

To find the power set of A, we need to find all possible subsets of A. The power set of A includes the empty set, which is a subset of every set, and the set A itself.

Here are all the possible subsets of A:

The empty set: {}
Subsets with one element: {1}, {2}, {5}
Subsets with two elements: {1, 2}, {1, 5}, {2, 5}
The subset with all three elements: {1, 2, 5}
Therefore, the power set of A is:

P(A) = {{}, {1}, {2}, {5}, {1, 2}, {1, 5}, {2, 5}, {1, 2, 5}}

The power set of A has 2^3 = 8 elements, which is the number of all possible subsets of A.

## Types of Power Sets we find in Problems
As if 2^N possibilities is not bad enough, many of the problems we run into are actually worse.

We'll take a look at examples but mainly the patterns that we run into. So no worries if you're not sure what the problem is 
just focus on the pattern that the solutions follow.

### Coin Change
You are given an integer array coins representing coins of different denominations and an integer amount representing a total amount of money.

Return the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return -1.

> You may assume that you have an infinite number of each kind of coin.

Input: coins = [1,2,5], amount = 5\
Output: 2\
We could pick: 
- {1,1,1,1,1,1}
- {1,1,1,1,2}
- {1,1,2,2}
- {2,2,2}
- {5,1}

The pattern is similar to the power set however the difference is that we can pick the same element multiple times.


### Combination Sum
Given an array of distinct integers candidates and a target integer target, return a list of all unique combinations of candidates where the chosen numbers sum to target. You may return the combinations in any order.

The same number may be chosen from candidates an unlimited number of times. Two combinations are unique if the 
frequency
 of at least one of the chosen numbers is different.

The test cases are generated such that the number of unique combinations that sum up to target is less than 150 combinations for the given input.

Input: candidates = [2,3,6,7], target = 7
Output: [[2,2,3],[7]]
Explanation:
2 and 3 are candidates, and 2 + 2 + 3 = 7. Note that 2 can be used multiple times.
7 is a candidate, and 7 = 7.

Again we can pick the same element multiple times but it still follows the power set pattern in some ways. A candidate can be 
in the set or not in the set based off some criteria.

