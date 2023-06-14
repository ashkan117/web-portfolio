---
title: "Power Set"
date: 2023-06-14T12:54:40-07:00
draft: false
---

## Introduction
In this case we want to try all possible variations of the given array. To be more specific we want to try all possible **combinations** of all possible **lengths**. 
What we're describing here is the **power set**.

There's a [great article](https://medium.com/outco/how-to-solve-power-set-c8ef7d1382ee) that elaborates even further on this however I made this post to
focus on the Python code as well as the itertools alternative code.

The power set of [1,7,2,4] would be:
```python
Start with an empty set: {}
Add the first element, 1, to all existing subsets: 
{}, {1}
Add the second element, 7, to all existing subsets: 
{}, {1}, {7}, {1, 7}
Add the third element, 2, to all existing subsets: 
{}, {1}, {7}, {1, 7}, {2}, {1, 2}, {7, 2}, {1, 7, 2}
Add the fourth element, 4, to all existing subsets: 
{}, {1}, {7}, {1, 7}, {2}, {1, 2}, {7, 2}, {1, 7, 2}, {4}, {1, 4}, {7, 4}, {1, 7, 4}, {2, 4}, {1, 2, 4}, {7, 2, 4}, {1, 7, 2, 4}

Our power set is all the above
{}, {1}, {7}, {1, 7}, {2}, {1, 2}, {7, 2}, {1, 7, 2}, {4}, {1, 4}, {7, 4}, {1, 7, 4}, {2, 4}, {1, 2, 4}, {7, 2, 4}, {1, 7, 2, 4}
```

## Recursive Implementation
The idea is fairly straight forward. We start with the empty list and at each stage we have two decisions to make. 
1. We create all possible subsets **including** this element
2. We create all possible subsets **excluding** this element

```python
def generate_subsets(index, subset, arr):
    if index == len(arr): return

    generate_subsets(index + 1, subset, arr)
    generate_subsets(index + 1, subset + [arr[index]], arr)

generate_subsets(0,[],[1,7,2,4]) # generates all the subsets shown above
```

The code itself is suprisingly simple so if you have a decent amount of experience with recursion
it is not the most complicated algorithm. The struggles with coming up with this implementation
comes from the inability to visualize how the recursion unfolds. 


## Itertools 
[Itertools](https://docs.python.org/3/library/itertools.html#itertools.combinations) has a lot of handy functions that you can use to replace the recursive solutions. 
The more comfortable you are with the recursion the easier it becomes to replace the work with an equivalent itertools function. 
In our case we're trying to generate the power set which requires you to undertstand that it is the same
as finding all combinations for all possible sizes.

```python
arr = [7,1,2,4]
for r in range(1, n + 1): # all possible lengths
    for subset in combinations(arr, r): # all possible combinations
        # combinations([7,1,2,4], 1) = [[7], [1], [2], [4]]
```

That's it! So instead of worrying about coming to the recursive solution or if your short on time you can 
leverage pythons library to do so.


## Time and Space Complexity
TODO

