---
title: "2d Matrix One Liner"
date: 2023-05-29T14:29:26-07:00
draft: true
---

dp = [[1]*n for i in range(m)]

# List Comprehensions
[num for num in nums]
[num * 3 for num in nums]

[[1]*cols for i in range(cols)]

The way you can imagine creating a 2D array is by creating the columns first then the rows.

We want 

To create a single row means we're creating a list with as many elements as we have columns.
[1] * cols 
If we want 5 cols 
this means 
[1] * 5 which in Python means create a list of 5 elements with a value of 1/

Now that we have our columns we'd want to create that same list for as many rows as we have.
We can read the 2D array as for each iteration we're creating a new row
[[1] * cols for _ in range(rows)]

1. Create the proper row first which depends on the number of columns
2. use List Comprehension to duplicate that row until we reach the desired number of rows.


## Sequence Repetition
https://docs.python.org/3.9/library/stdtypes.html#sequence-types-list-tuple-range

1. Generating a list with a single value
[1] * 5
Creating a list of 5 1s.
[1,1,1,1,1]
2. Padding a String
What if you want to have a consistent formatting in your CLI tool. Something like 
----------Hello----------

You can do 
"-" * 10 + Hello + '-' * 10

Note that this means the operation "-" * 10 returns a string.

3. Creating patterns
[1,2,3] * 3 
[1,2,3,1,2,3,1,2,3]
