---
title: "K Diff Pairs"
date: 2023-06-02T17:21:41-07:00
draft: true
---

# Introduction
We'll take a look at a problem similar to [k-diff pairs](https://leetcode.com/problems/k-diff-pairs-in-an-array/description/) in this post. 
Specifically we'll focus on a pattern that appears every now and then where you can improve your time complexity by changing the way you think.
These sorts of optimizations are important. Always start with the brute force method then take all the tricks you've previously ran into and 
the unique characteristics of this problem and see how you can take shortcuts. The brute force method works because it tries all 
possible combinations. **However, do you need to try all possible combinations?** Is there something special about this question that 
we can leverage that allows us to notice there's an easy way to wisely compute things.

# Brute Force
It asks for all possible pairs that have an absolute difference of k. The easiest thing you can do is try all possible pairs. 
This means that we're going to try (0,1), (0,2) ... (1,2) ... (n - 2, n - 1) where n is the size of the array. This code will look like
```python
for left in range(N):
    for right in range(left + 1, N):
        if abs(arr[right] - arr[left]) == k:
            count += 1
```

# Avoiding all possible pairs
Most of our complexity comes from the fact that we're searching to test which combinations of the 
pairs result in a difference of k. We don't actually need to do that. If we know one of the numbers and 
we know our difference we know which number we need to look for.

Input: nums = [3,1,4,1,5], k = 2

If we start at index 0, we see a value of 3. If we know the difference is 2 then the only number that it can be is 5.
This is just simple algebra, x - 3 = 2, therefore x = 5. The math is not important. The key insight is that 
we don't need to search to see if we have a number in our array that results in a difference of k in O(N) time.
We can just do that in O(1) time albeit at the cost of space complexity.

Let's update our algorithm.


