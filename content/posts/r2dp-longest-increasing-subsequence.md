+++
title= "Longest Increasing Subsequence (Part I)"
date= 2023-05-30T08:34:38-07:00
draft= true
series= ["Recursion to Dynamic Programming"]
+++

## Introduction
Following some popular leetcode solutions ([1](https://leetcode.com/problems/longest-increasing-subsequence/solutions/1326552/optimization-from-brute-force-to-dynamic-programming-explained/))

## Understanding the recursion through the choices we make

### Take or Don't Take
The way you can view the recursive solution is that should we :
1. Choose to take the current element and include it into our set of longest increasing
2. Choose not to take the current element

To be clear, you can only choose to take an element if its value is greater than its previous value. Otherwise we would
be breaking the fact that we're building a list of increasing subsequence. Regardless, by choosing to take a number
we're also changing what the maximum number we have seen so far is.

Let's take a look at this example
0   1 2 3 4 5  6   7
[10,9,2,5,3,7,101,18]
If we're investigating the subsequence [2] and we're now looking index 3 we can see that choosing the element with value
5, then it has a penalty. By that I mean that we're preventing the subsequence from being able to choose any element in the 
future that <= 5. That turns out to be too much of a penalty since choosing element 5 will cause us to miss the more optimal 
solution which is [2,3,7,101] or [2,3,7,18].

### Recursive Code 
```python
def lengthOfLIS(self, nums: List[int]) -> int:
    def dfs(i, prev):
        if i >= len(nums):
            return 0
        take = 0
        dontTake = dfs(i + 1, prev)
        if nums[i] > prev:
        take = 1 + dfs(i + 1, nums[i])
        return max(take, dontTake)

    return dfs(0, float('-inf'))
```

### Penalty forces us to try all possibilities
Taking a look at a few examples you see that you can't greedily determine which element to choose and be certain that its the right choice.
That is because of the penalty! When you pick an element its value becomes our new maximum element in the list as we saw this could 
lead us to a situation where if we pick an element that is too large too early then we miss out on all the smaller elements that 
the one we just picked.

### Disconnect between current choice and previous max
A very subtle detail is that this way of viewing the problem has a downside. Although it is very intuitive we're disconnecting
where this previous max came from. Recursively we just care about what the previous element was since we need it to determine 
if the current element we're looking at could be included or not since we must always satisfy the fact that we only have an increasing subsequence.
We'll visit this later when we solve the problem using a 2D array

## 2D DP Solution
### Initial Thought
Any easy way to work your way to a more efficient solution if you'd like to use DP is translating the states that your recursion relies on 
into a DP array. In our recursive solution it relies on
1. The current index we're on.
2. The maximum element we've seen so far.

Since we're using 2 states we can represent it with a 2D array to store our solutions to those subproblems. Let's say we wanted to do that.
What would the dimensions of our two dimensional array look like? Well the number of indexes is determined based off the length of the array.
The more interesting thing to notice is that the maximum element could be anything! The problem statement bounds the value of the elements 
to be -10^4 <= nums[i] <= 10^4. Trying to create an array of that size would be complete overkill since our original array could be tiny.
Even in the problem statement it says that the maximum number of elements our array can have is 2500. Worst case scenario we'd have a 2500 x 20,001 array.
If the array was small like we said then this could be a 5 x 20,0001 array to keep our states which again is overkill. [Solution II](https://leetcode.com/problems/longest-increasing-subsequence/solutions/1326552/optimization-from-brute-force-to-dynamic-programming-explained/
) talks on this more. **Key takeaway is that we want to store the previous element we've seen as an index so that we can bound our two dimensions relative
to the size of the nums array**


## Resources
https://leetcode.com/problems/longest-increasing-subsequence/solutions/1326552/optimization-from-brute-force-to-dynamic-programming-explained/
https://www.youtube.com/watch?v=xRv-tnKY4HU
