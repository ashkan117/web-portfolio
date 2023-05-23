---
title: "Beware of blind submitions"
date: 2023-05-19T10:19:14-07:00
draft: true
---

## Introduction
I ran into this problem for an initial assessment and found myself short on time. I was comfortable with a fairly similar problem in [container with most water](https://leetcode.com/problems/container-with-most-water/)
however wasn't sure if the way I was "extending" the problem was correct. Unfortunately after finding the problem on leetcode I saw that I did not do it right.

## Why assessments are harder than the way you practice
If you haven't ran into the problem you are being interviewed on it is crucial that you have a list of similar problems in your toolbelt. I believe that if you 
are doing leetcode problems and truly understanding them it is usually not hard to extend those problems to other similar problems. Those problems in your 
toolbelt give you the insane advantage of not worrying too much about the coding aspect and allow you to focus on the problem.

Assessments hide test cases from you which means that you can't blindly come to a solution without being able to prove it. You can't try a naive approach then submit it 
to determine if you did it right or not. These **blind submissions** are a horrible practice that I was doing without really realizing.

## Soltutions
1. Edge cases. Edge cases. Edge cases.

I will elaborate on why mine successfully worked on the non hidden test cases but failed on the hidden ones. There were a set of problems that would have helped me realize
that my approach was flawed.


2. Make sure that when you're doing leetcode you can prove the logic behind your algorithm before submitting your practice problems.

Being short on time I wasn't able to focus much on the problem and my naive approach was not correct. Although it passed the non hidden test cases being unable.

## Problems I had

## Solving the simpler problem
Let's not worry about finding the maximum area that a section holds. Let's just think about how much the entire array holds.



### Code



## Alternative Problem
The problem that I was asked to solve was a similar problem but not exactly like the [leetcode one](https://leetcode.com/problems/trapping-rain-water/solutions/). 
The one I was solving just asked for the **largest** amount of water that can be trapped. Leetcode's problem asks for the **total** amount of water that can be trapped.

## My initial Naive Approach
Unsure of how I can extend the container problem I sought to just solve the problem in a brute force way. My approach was to try all possible combinations of 
the "container" then account for the fact that we would have obstactles in the way which are the buildings.


## Code Brute Force

```python
def trap(self, height: List[int]) -> int:
        N = len(height)
        max_area = 0
        for left in range(N):
            for right in range(left + 2, N):
                lh, rh = height[left], height[right]
                width = right - left - 1
                area = min(lh,rh) * width
                print(lh, rh, width)
                water = 0
                for i in range(left + 1, right):
                    # print(f'found a height of {height[i]}, decrement area')
                    # area -= height[i]
                    if height[i] <= min(lh, rh):
                        print(f'found a place water can fill add to water with height of {height[i]}')
                        # water += height[i]
                        water += min(lh, rh) - height[i]

                
                print(f'this size can hold {water}')
                max_area = max(water, max_area)

        return water
```
