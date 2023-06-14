---
title: "Non Divisible Subset I"
date: 2023-06-14T12:43:16-07:00
draft: false
---

## Introduction
Hackkerank medium problems continue to be a challenge for me. In this problem that we'll take a look at it was challenging due to the fact that the 
brute force method seemed fairly involved and the inability to make connections to solve this problem in a faster way. In this post we'll take a look
at the brute force solution.

1. Generate all possible combinations of the array (Generate the power set)
2. Sort based off length of array, largest first (As soon as we see one that is valid we can just return)
3. Validate constraint for all sets in the power set

In this post we'll be focusing on the validating the constraints.
Generating the power set is not as hard as it seems if you haven't done it before. Granted you do need to be fairly comfortable with
recursion. You can take a look at my [power set article]({{< ref "power-set.md" >}}) to see how we can do this. It's important to note though that we're 
looking at combinations. That is because order does not matter. [1,7,2] is the same as picking [1,2,7] in our book.

## Checking the constraints
So given a some set how can we show that it validates the constraint of this problem that no two elements in the set can sum to a number 
that is divisible by some number k.

If our array is [1, 7, 2] and our k = 3, this won't be valid.\
1 + 7 = 8 (8 is not divisble by 3)
1 + 2 = 3 (3 is divisble by 3, this array is not valid) 
1 + 7 = 3 (9 is divisble by 3, this array is not valid)

The way we can view this is that we fix one element and check if the sum with all the other elements 
is divisible with k or not. An easier way to put this is that we're trying all possible pairs of the array. That's just a 
nested loop.
```python
def is_divisible(subset, k):
    for i in range(len(subset)):
        for j in range(i + 1, len(subset)):
            if (subset[i] + subset[j]) % k == 0:
                return False
    return True
```

```python
def is_divisible(subset, k):
    for i in range(len(subset)):
        for j in range(i + 1, len(subset)):
            if (subset[i] + subset[j]) % k == 0: return False
    return True

def nondivisible_subset(n, k, arr):
    max_size = 0
    for r in range(1, n + 1):
        for subset in itertools.combinations(arr, r):
            if is_divisible(subset, k):
                max_size = max(max_size, len(subset))
    return max_size
```

## Conclusion
However to conlude this post let's take a look at the time and space complexity. 

The two things that our algorithm does is generate the power set and for each of those sets in the power sets we investigate if it's divisible.
The time it takes to check if a candidate is divisible is O(N^2) where N = len(array). The time it takes to generate the power set is
O(2^N) since for every element there's a path where that element is in the set or it is not. For more details on the power set please check out 
my post on it or the one from [Sergey](https://medium.com/outco/how-to-solve-power-set-c8ef7d1382ee). So within our nested loop that generates the power set
we do the O(N^2) time to check if it's divisble. This leads us to a time complexity of O(2^N * N^2).

The space complexity is easier. It requires as much space to store the power set. This also ends up being O(2^N) since the power set will generate 
that many sets. We store every possible variation that occurs with a given array. At each iteration we double our work which leads to the exponential 
space complexity.

The brute force solution ends up being a little involved but ultimately not too bad. In the next post we'll take a look at the more optimal solution. 
