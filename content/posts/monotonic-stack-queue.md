+++ 
draft = false
date = 2023-05-23T12:49:41-08:00
title = "Monotonic Stack/Queue (Part 1)"
description = "General introduction to Monotonic Stack with the example of Next Greater Element"
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

## Goal
Introduces you mainly to the Monotonic Stack in an intuitive way with code snippets included. This post should help you understand when to start thinking of using a monotonic stack.

## Introduction
In some problems like [next greater element](https://leetcode.com/problems/next-greater-element-i/description/) you see this pattern where one value can trump many others. Let's look at an example of this relative to next greater element. 

[3,2,1,9]\
In this example we see that all the elements (3,2,1) end up all sharing the next greater element. Intuitively I think the easiest example to understand is if you're in a line of people. If you can only look ahead you'll realize that the next tallest person in front of you is
1. Visible
2. Blocks you from seeing people in front of them

![Standing in Line courtesy of AlgoMonster](/images/monotonic-stack-line.jpg)


This idea that your vision is blocked by a "leading" element is the use case where monotonic stacks/queues shines. This is because it allows us to **report back** to each index in the array where your next greater element is.

## Intuition
The reason why a stack is useful is because we're **building a line** of elements that can be **ejected** as soon as some constraint is met. Again let's focus on the next greater element 
example in order to give more concrete examples. 

[3,2,1,9]\
So in this case the line that builds is (3,2,1) because they are all waiting for some element to appear that is larger than them in order 
to answer the question of what is my next larger element. Why use a monontonic stack? To make our algorithm faster, by building this line we're 
able to answer some question for multiple elements at once. In thise case we're asking the question what is the next element in the list that is greater than me.
We end up building a line in our stack however as soon as we see 9 we have enough information to answer who is my next greater element for (3,2,1) all at once.

This idea of building a line goes hand in hand with queues and stacks which is why we use them for this algorithm in order to increase our time complexity. 

> We'll focus on using a stack in our examples. This is because there are seemingly a lot more examples for it however when I run into a queue I will add those examples.

### General Code
The code is actually really simple however there are a lot of variations since the condition and the information gained is relative to the question.
However, just think of it as we're iterating through some list where we are using the stack to extract some information
in an efficient manner. We always append an item but before we do we must make sure that the constraint is not violated. 
```python
stack = []
for item in items:
    while stack and item <= stack[-1]:
        top = stack.pop()
        # information gained here
        # at this point we've learned that 
    stack.append(item)
```

For a **increasing** stack the variant is that all elements in the stack, if any, must be smaller than (or equal to) us.\
For a **decreasing** stack the variant is that all elements in the stack, if any, must be greater than (or equal to) us.

To keep these variants satisfied we kick out any elements in the stack that violate our condition.

> The easiest way to reach the code is by thinking backwards. What case leads us to a situation where the condition is not satisfied. In the next greater element case,
this would be if we never have a next greater element. If our input is [5, 4, 3, 2, 1] then our output is [-1, -1, -1, -1, -1] since each element is its own relative max.

## Report Back Functionality (Next smaller element)
I believe one of the reasons that this concept is hard to understand is because the problems that rely on it don't make it intuitive. I was trying to think of a 
practical example where we can use the monotonic stack to find the next smaller number which was a little hard. However, lets take a look at the following example.
Let's say we have an array that represents how level a road is. 1 means level, 0 means pothole. So [1,1,1,0,1,0] would mean our road has two holes in it.
We can use (non-strict) decreasing monotonic stack to find the holes. 

This led me to wonder, why can't we just iterate forward to find the holes? The only answer I can think of is that we're able to report back to 
each element where the hole is. We can return back an array that represents the index of the next hole. 

### Break Down
In this case we're interested in reporting back the index of the holes/potholes. This means in our stack we keep track of the indicies and check the values
through those indicies (items[stack[-1]])
So while we see a level road we keep on adding to the stack. As soon as we see a hole then we want to report back to all those indicies of the road
that we found a hole at some index.

### Code
```python
stack, result = [], []
items = [1,1,1,0,1,0]
for i, item in enumerate(items):
    while stack and item < items[stack[-1]]:
        top = stack.pop()
        # information gained here: we've found a 0
        # pop off as many ones as we have in the stack
        result.append(i)
    if item == 1: stack.append(i)
    else: result.append(-1) # holes don't care about other holes
while stack:
    stack.pop()
    result.append(-1)
print(result) # [3,3,3,-1,5,-1]
```

## Conclusion
This pattern is extremely general but since the code is not too involved it's fairly 
easy to implement as long as you understand the motivation.

Whenever you see the following ideas in a question think to use Monotonic Stack/Queue
1. There is an item in a list that **shadows/trumps** other elements
2. Report back to the other elements which value is the one that shadows the others in an efficient manner

## Resources
https://algo.monster/problems/mono_stack_intro
