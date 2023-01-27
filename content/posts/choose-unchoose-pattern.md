+++ 
draft = false
date = 2023-01-26T19:15:28-08:00
title = "Choose Unchoose Pattern Case Study"
description = "A deep look at two different implementations of the choose unchoose pattern for backtracking"
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

## Pre Requisites 
This post won't elaborate on the following topics since there is already many great ones:
- Choose unchoose pattern
- Leetcode Combinations Problem https://leetcode.com/problems/combinations/description
- Backtracking

## Context
This pattern is often just given as a part of a template for similar problems/
https://leetcode.com/problems/combinations/solutions/429526/general-backtracking-questions-solutions-in-python-for-reference/ 
https://leetcode.com/problems/combinations/solutions/27006/a-template-to-those-combination-problems/

My goal is to better give intuition behind it by comparing it to an equivalent but different code

## My Initial Solution
I was working on Leetcode Combinations problem and realized that this is an choose unchoose pattern like problem but my implementation seemed a little funny in comparison.
https://leetcode.com/problems/combinations/description/

There were three main differences:
1. My recursive function contained two recursive calls while theirs contains one
2. I did choose and unchoose while there's seemingly had that be implicit. (Resulted in the depth changing while theirs remained the same)
3. Mine contained an additional base case

We'll see that the main difference ends up being since we use a loop in one but not the other



## Two Recursive Calls vs One
choices = [1,2,3,4] which is the array we want to find the combinations of dependent on k. In this case the combinations are [1,2], [1,3], [1,4], [2,3], [2,4], [3,4]

```python
path.append(choices[i])
dfs(i + 1, depth - 1, path)
path.pop()
dfs(i + 1, depth, path)
```

```python
for i in range(index, len(nums)):
    dfs(depth-1, i+1, path+[choices[i]])
```

### Who's making the decision
I think it's a little neat that the second solution is telling the dfs algorithm more information while in the first code block we see that the dfs algorithm picks what to do next. Looking at this code it almost looks like there's no way they produce the same results, yet they do.

The decision making is the thing that varies. The one with the loop starts with the [] base case and it builds the tree from there.

### The lack of control results in additional work done
The one without a loop starts with the [] base case and will build on it to find solutions but note that it will end up in with the [] case many times. Going my approach I printed out the call stack. 
The method with the loop has a slightly different approach. It will start with the [] base case but after the lap the loop will give the next recursive calls their starting points.\
[]\
[1] [2] [3] [4]

It will avoid going back to the [] case

I've included some scratch at the bottom of the screen to illustrate the method call with no loop.

## How does theirs choose and unchoose 
This point ends up being more language specific than you might think. The reason we have the append and pop is since we're passing the path variable to the recursive call while the other one does not change the path variable itself. Let's look at an example where path = [1]

```
path.append(2) # path = [1,2]
dfs(i+1, depth - 1, path) # path = [1,2]
path.pop() # path = [1]
dfs(i+1, depth - 1, path) # path = [1]
```

On the other hand the other solution relies on what's evaluated it sent to the future stack but this stack remains untouched for future calls. You'll notice the difference on each iteration of the loop
```
# path = [1]
for i in range(index, len(choices)):
    dfs(depth-1, i+1, path+[2]) # dfs(depth-1, i + 1, [1,2]) is called
    # path at this point is still [1] just like mine
    # on next iteration of the loop it will send dfs(depth-1, i + 1, [1,3]) is called
```

### Why does my code require an additional check?
Finally we come to the third difference. This is a bit of a reoccuring theme for me where I get off by one errors or out of bounds errors. Understanding your explicit range of numbers is really important. It could be subtle but
still really important since it helps you get more of a concrete feel of edge cases and how your code iterates through the solution.

I thought mine would not need the additional i >= len(choices) check since the depth base case will handle it. Not true when you're trying to build the path with the last element in the choices array\
choices = [1,2,3,4]\
path = [4]

At this point it has not reached the depth of two so it will try and pick the next element in the choices array where it doesn't exist.

The loop avoids this since 
```
for i in range(index, len(choices)): # clear bounds of [1,len(choices)]
    # will only call dfs recursive function with these explicit numbers
```
len(choices) = 4 and our index is on the last element (index = 3, value = 4) it will have one last recursive call. We're passing this recursive call an index of 4 which will now cause the loop not to execute.
```
for i in range(5, 4): # out of bounds
    # not executed
```





# Scratch
## Lack of control results in additional work done
choices = [1, 2, 3, 4]\
(index, path)\
index we're looking at from choices array while path which represents a snapshot of a single combination that we're building\
0 []\
1 [1]\
2 [1, 2]\
2 [1]\
3 [1, 3]\
3 [1]\
4 [1, 4]\
4 [1]\
1 []\
2 [2]\
3 [2, 3]\
3 [2]\
4 [2, 4]\
4 [2]\
2 []\
3 [3]\
4 [3, 4]\
4 [3]\
3 []\
4 [4]\
4 []\
