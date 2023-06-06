+++
title= "2D Matrix One Liner"
date= 2023-05-29T14:29:26-07:00
draft= false
tags=["python-tips"]
+++

## Introduction
Python can be a powerful language since it allows you to acheive a lot through a few lines of code. The example we're taking a look at in this post is how 
to create an M x N matrix.

```python
dp = [[1]*n for i in range(m)]
```

We'll break this into two parts:
1. Sequence Repetition 
2. List Comprehensions 

## Sequence Repetition
https://docs.python.org/3.9/library/stdtypes.html#sequence-types-list-tuple-range

Addition is typically just used for numeric operations however Python allows you to use the '*' operator for sequence types as well.
**As a refresher sequence types can mean list,tuple,range,str,bytes,bytearray,etc**.

The * operation with sequences is viewed as repetition. Given some sequence repeat that pattern some number of times. Taking a look at some examples 
makes this easier to see. The repetition operation can be used to:

1. Generating a list with a single value
[1] * 5
Creating a list of 5 1s.
[1,1,1,1,1]
2. Padding a String
What if you want to have a consistent formatting in your CLI tool.\
Something like "----------Hello----------" which you can do by 

```python
"-" * 10 + 'Hello' + '-' * 10 
# Note that the result of this operation is a string
```

3. Creating patterns
```python
[1,2,3] * 3 
# [1,2,3,1,2,3,1,2,3]
'9875' * 3
# '987598759875'
```

Therefore in our matrix example we see that we're initializing the rows to have a value of 1.
Note that the repetition operation is for sequences which is we we do [1] * 5 instead of 1 * 5. 
The latter is just regular multiplation.

## Multi Dimension List Comprehensions

The final part that we reach is understanding the order of the iteration that must be done to handle nested loops with list comprehensions.
Single loops are easy to replace with list comprehensions and they are still really readable.

```python
# create a copy of the nums array
[num for num in nums]
# create a copy of the nums array where each value is 
# multiplied by three compared to the original
[num * 3 for num in nums]
```

Let's look at the more verbose code of our list comprehension.

```python
dp = []
for row in range(num_rows):
    current_row = []
    for col in range(num_cols):
        current_row.append(1)
    dp.append(current_row)
```

What we're doing is for each row we're creating an empty list. Then we add 1 to that list for as many columns as we need.
This means if we have 7 columns we end up with [1,1,1,1,1,1] after our inner loop ends and then we add that list to our 
dp list which results in [[1,1,1,1,1,1]]. We repeat that work once for every row we need which results in a 
ROW x COLUMN matrix.

How can we replace that with list comprehensions? Well just like the example above **we create a single row by iterating through
the number of columns we have**.

To create a single row means we're creating a list with as many elements as we have columns.
```python
[1] * cols 
# [1] * 5 = [1,1,1,1,1]
```

I believe the last piece of confusion comes from understanding how the appending of each item in 
a list comprehension works.

```python
[expression for _ in arr]
```

Python will evaluate that expression and append that to the list we're building. Note that we don't have to use the item in the array however
the way that the iteratable affects the list comprehension is the number of items we're appending. In the case above we're creating a list 
that is the size of the length of arr.


**For the 2D array we want a list of lists therefore the expression must evaluate to a list**. This leads us to 

[[1] * cols for _ in range(rows)]

1. Create the proper row first which depends on the number of columns (Using Sequence Repetition)
2. Duplicate that row until we reach the desired number of rows. (List Comprehensions)

## Final Remarks
This is not the only way to create a 2D array however I thought it is a fairly short way to do so. The concepts
behind them are not too complicated and its good to know some of the details that it takes to come up with a one liner like this. 
One liners are cool but not always worth it so use them wisely.


