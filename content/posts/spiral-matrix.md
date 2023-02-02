+++ 
draft = true
date = 2023-02-01T08:57:19-08:00
title = ""
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++


# Discovery Stage
## Understanding the return condition
Although it's always important to do determine what problem you're trying to solve is; it's especially crucial in the beginning process of leetcode. Mainly due to the reason you don't want to solve a problem that is harder than the one the question is asking you to do. Understanding how to iterate through the matrix in this spiral fashion is much easier than sprilang through the matrix and maintaining the matrix if you had to do things in place.

## Trying to recognize the pattern through brute force
Once you realize you just need to see that the problem is asking you how to iterate through the matrix things become less daunting. You just need to discover the pattern.

### General Pattern
1. top left to **top right**
2. **top right** to *bottom right* 
3. *bottom right* to **bottom left**
4. **bottom left** to top left

### Realizing you need another loop for the processing logic
You can see early on that you can move a cell in all directions(left, right, up, down). However you can also notice that in the example cases given you move right twice and all other directions once. Understanding how to code that ends up being the tricky part. Maybe that is the pattern, move right, down, left, up, right one last time? That is not the case. If you think of a big matrix (5x5) then you could see that after you strip the outer layer away you get a 3x3 matrix so you can iterate one more time.
How do we start on this second iteration of the loop?

1 1 1 1 1\
1 2 2 2 1\
1 2 2 2 1\
1 2 2 2 1\
1 1 1 1 1

First iteration:
You start with row 0 and you go to row 4
You start with col 0 and you go to col 4

Second iteration:
You start at row 1 and you go to row 3
You start at col 1 and you go to col 3

**This implies that the starting point and ending point for both row and col are variable**


## What type of matrix is needed to be able to spiral
Can you do this with a 2x2? Yes you can. This poses the question of how do we deal with the odd shaped matricies. 

# Handling odd shaped matricies (Failed here)
This is where I had problem with this problem. After discovering the main way to iterate through the matrix I wasn't sure how to handle odd shaped matricies. This was mainly because it's a little hard to understand how one iteration of the loop changes the subproblem. 
What our algorithm is doing is stripping away the outer most layer. So if we have a 3 x 3 matrix we will visit 8 cells after running the code inside the while loop. This makes the next iteration solve the 1x1 problem.
If the matrix is 4x4, the next subproblem is 2x2
The reason behind this is that our loop will go through two rows and two cols.
Our main chunk of work processes the outer layer of a matrix. This effectively is the same as processing two columns and two rows. So if we have an M x N matrix, after one iteration we still have a M - 2 x N - 2 to process.

Another way you can view this is by counting how many elements you process each iteration and notice the pattern there. If it's a 3 x 4 array we have 12 elements then we'll visit 4 elements while going from top left to top right. 2 elements going from top right to bottom right. 3 elements going from bottom right to bottom left. Finally 1 element going from bottom left to bottom right. This means we visit a total of 10 elements, we have two left to visit(remaining is a 1x2 matrix).

## Claim 1 (After our outer loop ends, if a matrix remain inside it is either a 1xn or a mx1)
With this knowledge see if we have any insight on the 3 x 4 case.\
1  1  1  1\
1  6  7  1\
1 1 1 1

### Understanding when your loop ends and its implications
Assuming we want to end the matrix early (there is a way to solve this problem by ending it late and removing the elements that are extra later); we end our array when our top and bottom pointers are equal to OR left and array pointers are equal to. 
Our loop has two pointers for each ends of the matrix. left, right, top, and bottom. After each iteration both ends grow closer together, meaning that left and top grow by 1 and right and bottom grow by one. Like we said the loop processes two rows and two columns so this makes sense.

So given any matrix we will contine withour loop until we reach one more row to process AND/OR one more column to process. Take 5 x 4 matrix for example, we start with 
1 1 1 1\
1 2 2 1\
1 2 3 1\
1 2 3 1\
1 1 1 1\
left = 0, right = 3\
top = 0, bottom = 4

After 1 iteration\
left = 1, right = 2\
top = 1, bottom = 3\

After 2 iterations\
left = 2, right = 1\
top = 2, bottom = 2\

Since it's a non square matrix we know we have more elements to process.
We see that the left and right condition are now no longer left < right so that means we have elements in a column fashion to process. In our array we can see that the elements with the value 3 have not been processed. We want to process [1][2] and [1][3] 
range(left, right)

# Handling the work needed to be done after the loop ends
If we need to 
## Ending with the right amount of work to be done
### Order of the nested loops matter (In the edge case we only want to perform one of the loops)
```python
for i in range(left, right):
    res.append(matrix[top][i])
top += 1
for i in range(top, bottom):
    res.append(matrix[i][right - 1])
right -= 1
if not (left < right and top < bottom):
    break
```

## Switching things to add more elements than needed 
As we can see we know we have more elements to process but we don't know if rows and columns 


This means that for non square matricies, we end up knowing we either have more rows to process or more 


After one iteration we get a 1 x 2 matrix
6 7 

These two elements are the only ones that still need to be processed

https://leetcode.com/problems/spiral-matrix/solutions/394774/python-3-solution-for-spiral-matrix-one-of-the-most-easiest-you-will-never-forget/

## Claim 2: No added work needs to be done for smaller matricies
1 
2


