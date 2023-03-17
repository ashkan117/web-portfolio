---
title: "Choose Unchoose Pattern II"
date: 2023-03-06T09:58:35-08:00
draft: daft
---

## Introduction
This is part two of my discussion on the choose unchoose pattern found in programming interview questions. This post will specifically focus on the loop aspect of the pattern. We will use the [Combination Sub](https://leetcode.com/problems/combination-sum/description/) problem as the application to the concepts.

## Understanding why a loop also works
I think the easiest way to view the loop and how it works is by understanding that it will try the first candidate as much as possible and when that fails it will **iterate to the next viable candidate.**

Let's say our input for the problem is [2,3,6,7], target = 7

The loop will only iterate to the next step once the recursion reaches a halt. In this case it will continuously add 2 to the path until it finds a solution or until the path is invalid.
[2] target = 5\
[2,2] target = 3\
[2,2,2] target = 1\
[2,2,2,2] target = -1

At this point the recursion hits the base case where target is now not valid since target < 0. It will return to the last previous valid case. [2,2,2] target = 1

We started with i = 0 and nums[0:] means the whole array is still valid. Once we hit the invalid base case we can now finally iterate to the next candidate. The loop does this by iteration.
nums[1:] will now remove the first element as a valid candidate and try the next one. This means the next thing that is tested is [2, 2, 2, 3].

```python
for i in range(len(nums)):
    # path is [2,2,2]
    choice = nums[i]
    dfs(nums[i:], target-nums[i], path+choice)
    # code below is implicit
    # if choice is valid or choice is not valid:
    #     continue
```

> Iteration removes the current candidate from the solution space

Once we iterate we're looking at a shrunken candidate list. We removed one candidate from that list and now will look at the new candidate until we reach a valid solution or until it is no longer valid.


