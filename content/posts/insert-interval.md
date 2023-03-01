+++ 
draft = false
date = 2023-03-01T09:10:54-08:00
title = "Insert Interval"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++


# Introduction
Today we'll take a look at [Insert Interval](https://leetcode.com/problems/insert-interval/description/). The reason why I think this is a useful problem to learn is because it helps you approach these problems in such a way where the code is more intuitive. Essentially the reasoning behind this problem and others like it is that it is not the high level solving that's difficult but rather the code. The way of viewing these problems that simplify the thinking is by viewing it as a **finding an index**. These sorts of problems are called [line sweep](https://leetcode.com/discuss/study-guide/2166045/line-sweep-algorithms) 

I wanted to focus on the approach of searching in this post rather than the interval and merging aspect. I may make a post about that in the near future. Also we're solving this problem "out of place" meaning we will not touch the original intervals array since it gets messier worrying about the changing size of the list.

# Finding the index of the insert
Once you see this problem as searching for an index of where to insert it's a lot easier to reason about. We just need to put into words how we find the index of this problem. Let's look at the following example case to help.
>Input: intervals = [[1,2],[6,9]], newInterval = [3,5]\
>Output: [[1,2], [3,5], [6,9]]

In this case we see that there is no merge happening. We can view this as We're iterating through the intervals list and asking ourself can we place the newInterval after the current element we're looking at. If the current_interval finishes fully before our newInterval starts then we know it should placed at that index. 

When we compare (x,2) and (3,y) we see that (x,2) should be placed first because the end of the current interval is < the start of our new interval. **There is no possibility of merge here**
Next we compare (x,9) and (3,y) we see that the new interval starts before the current interval ends. **This case could have a merge**. We know that x = 6 and y = 3 but let's pretend we didn't. What would we learn about this case.

Case 1: The x and y really determine whether a merge will happen or not. If x > y then we can see no merge happens.This would be like our current case (6,9) and (3,5).\
Case 2: These two intervals are disjoint and therefore our new interval can be placed fully before the current interval without worrying about merging.
However if x <= y (5,9) and (3,7) means we merge. 

> Tip: Viewing the intervals makes it much easier (Credit to Emre Bolat for the pictures, link in resources)

![image](/images/interval-merge-ii.png)
Think of A as the current interval and the new interval as B. In example I it's like case 1 since newInterval does not merge and comes after the current interval. Case 2 is more like 3) since we do merge and additionally we will fully merge.

> Observation 1: we're looking for the first index where there is a chance of merging. newInterval[0] <= current_interval[1] (the start of our new interval falls between the current interval's end)

## Code Part I (Find insertion point)
To code this we can just move to the point where the end of the current interval has no conflict with the start of our new interval. 
```python
i = 0
while i < len(intervals) and intervals[i][1] < newInterval[0]:
    # current interval's end = intervals[i][1]
    # new interval's start is = newInterval[0]
    res.append(intervals[i])
    i += 1
```

# Do we insert or merge
At the end of this loop we're at a point where we want to insert this interval but there is a possibility of merges as we saw in the example cases. One way we can view the insertion
in this case is to see it as a **merge**. 
>Input: intervals = [[1,3],[6,9]], newInterval = [2,5]\
>Output: [[1,5], [6,9]]

>Input: intervals = [[1,3],[6,9]], newInterval = [2,7]\
>Output: [[1,9]]

We see that we start with N number of elements and in the merge case we will have N or less elements at the end. We see that everything could merge into a single interval too so there is no real need to insert any new elements.
In the second example we see that we might need to merge multiple times. The way you can view things conceptually is if we notice a merge we're on hold since we could merge again. So if we do see that merge
our code goes hang on, the interval we want to place is actually wider. For example (1,3) is what we wanted to place but we see that (2,7) merges. So instead of placing (1,3) we could place is (1,7).
The reason I say "we could" instead of "we will" is that you don't know if the element after this would merge too. (6,9) will merge with (1,7) in this case so instead of placing (1,7) just wait until you're done merging
then place the interval. We see that our (1,7) becomes (1,9) and since there are no more elements with the possibility of merging we can finally place (1,9).

## Code Part II (Do we merge? If so how much? Bubble the merges)
```python
while i < len(intervals) and intervals[i][0] <= newInterval[1]:
    newInterval[0] = min(intervals[i][0], newInterval[0])
    newInterval[1] = max(intervals[i][1], newInterval[1])
    i += 1
```

# Insert New Interval (In the case of merging, newInterval = mergedInterval)
We've finally found the correct place to insert our newInterval, note that in the case of merging we just update newInterval to be the result of **coeallesing**/bubbling all the merges.
Insert the newInterval/mergedInterval and now we're at a point where we are certain no more merging could happen so insert the rest of the elements

## Code Part III (Finally insert newInterval/mergedInterval + remainder)
```python
res.append(newInterval)
while i < len(intervals):
    res.append(intervals[i])
    i += 1
```

# Analysis
Once this loop ends we've done our "sweep" and could now handle the interesting case of 
Reverse Linked List II


# Resources
https://emre.me/coding-patterns/merge-intervals/
https://leetcode.com/problems/insert-interval/solutions/1581680/python-easy-solution-two-approaches/
