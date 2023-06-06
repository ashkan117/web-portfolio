+++
title= "Reversing rows and cols with Python"
date= 2023-06-03T10:29:24-07:00
draft= false
series= ["Flip Matrix"]
tags= ["recursion", "backtracking", "python-tips"]
+++

## Introduction
I want to cover the [Flipping the Matrix](https://www.hackerrank.com/challenges/flipping-the-matrix/problem) problem in a non optimal way 
in order to practice a few fundamentals that I think are important. In this post we'll cover nested loops and list comprehensions and in the next post we will
cover backtracking and combinations.

List comprehensions are not a one size fix all tool. The more complicated the looping logic is the harder it becomes to read the code in many cases.
Use it where it feels natural and don't force things. Even in this case there is not an insane benefit to using them but I thought it would 
be good educational content.

In this problem list comprehensions end up being very handy in this problem since it's an easy way to create a matrix
using our original matrix but not modifying the original matrix.

> When manipulating existing iterables see if list comprehensions can simplify your work and logic.

## Using the idea of a compliment to simplify code
When we're flipping/reversing a row/column what we're really doing is keeping everything the same except for the row or column we're interested in.
For that row or column we're interested in we want to look at things in reverse. 
However, that sort of logic is not too simple to code since we're trying to do conditional loops. It would look something like. 

```python
if row_we_want_to_reverse:
    iterate in reverse
else:
    iterate regularly
```

This type of logic could be simplified by viewing things in a different light. We'll look at the "iteration pattern" through the following example:

[1, 2, 3],\
[4, 5, 6],\
[7, 8, 9]

If we want to reverse the second row what indicies would that be?

(1,0), (1,1), (1,2)

If we wanted to reverse that row we really would want the value of 

(1,0) -> (1,2), (1,1) -> (1,1), (1,2) -> (1,0)

So when the column is 0, we want it to point to the last element in the array. As the 
original columns index increases, the column's value we want to swap it with becomes smaller.

This pattern is fairly useful to recognize. Starting at the last index of our array translates to 
num_columns - 1. As we iterate through our column index in an increasing fashion we can use that 
to have our desired value index approaches 0.

```python
def reverse_row(row, m):
    rows, cols = len(m), len(m[0])
    # new_matrix = [[m[r][c] for row in rows] for col in cols]
    # make a copy of this row, for each row in matrix
    new_matrix = [row[:] for row in m]
    for r in range(rows):
        for c in range(cols):
            # if this is the row we're interested in
            # only modify this row, everything else stays untouched
            if r == row:
                new_matrix[r][c] = m[r][cols - c - 1]
    return new_matrix
```

## List comprehension to reverse row
We can shorten this even more by using list comphrehensions. Let's start simple by imagining we wanted to just create a new copy of the 
matrix using list comprehensions. 
1. We need a list within a list since it's a 2D matrix. ([[]])
2. We want the element to match the matrix m's values. ([[m[r][c]]])
3. Last step is to define the row column indicies order of things. For a multi dimensional list comprehension you need to start from the inner loops and work your way out.
(The inner most loop starts with the column iteration. Then its the row iteration)

```python
def copy(row, m):
    # make a copy of this row, for each row in matrix
    return [[m[r][c] for c in cols] for r in rows]
```

Now let's update this to reverse a given row. The only thing that changes is that if we reach the row we want to reverse we need 
to instead place the value in its compliment column index.
```python
def reverse_row(row, m):
    # make a copy of this row, for each row in matrix
    return [[m[r][c] if r != row else m[r][cols - 1 - c] for c in cols] for r in rows]
```

> Don't worry too much about the list comprehension details if it doesn't make sense. You can 
> always do it the more verbose way. List comprehensions get a bit too complicated when it comes to
> multiple dimensions of an array.

## List comprehension to reverse column

Finally lets reverse the column following similar logic from reversing the row. In this case now the column index actually stays
fixed while we iterate through the rows. And again to avoid the weird conditional looping idea we can just use the "compliment".

```python
def reverse_column(col, m):
    rows, cols = len(m), len(m[0])
    new_matrix = [row[:] for row in m]
    for r in range(rows):
        for c in range(cols):
            if c == col:
                new_matrix[r][c] = m[rows - 1 - r][c]
    return new_matrix
```

One last time using list comprehensions.

```python
def reverse_column(col, m):
    rows, cols = len(m), len(m[0])
    return [[m[rows - 1 -r][c] if c == col else m[r][c] for c in range(cols)] for r in range(rows)]
```

## Final Notes
List comprehensions come in handy to make your code more compact however it could make it harder to read if the logic is too complex.
While building your list comphrehensions it could be easier to build if you do things iteratively. In our case we pretended 
we just wanted to copy over the array at first. Once that feels comfortable you can manipulate the values of the sequences you're 
iterating over.
