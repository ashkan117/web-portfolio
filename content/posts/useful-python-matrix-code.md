+++ 
draft = true
date = 2023-02-01T17:32:36-08:00
title = "Useful Code for Matrices"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

https://leetcode.com/problems/valid-sudoku/description/
# How to iterate through chunks
When solving the following problem you also need to solve it for multiple subproblems. 
Determine if a 9 x 9 Sudoku board is valid. Only the filled cells need to be validated according to the following rules:

Each row must contain the digits 1-9 without repetition.\
Each column must contain the digits 1-9 without repetition.\
Each of the nine 3 x 3 sub-boxes of the grid must contain the digits 1-9 without repetition.

## Handing it for the top left 3x3 matrix
This means if we're given a List[List[int]] that represents a 9x9 matrix we want to solve it for all valid 3x3s
First three rows, first three columns\
(0,0), (0,1), (0,2)\
(1,0), (1,1), (1,2)\
(2,0), (2,1,) (2,2)

There's no shortcut to get the first three rows. The way you solve this is for each row you get the first three columns, do this three times
square = [board[0][i] for i in range(3)]

To do this for the first three rows and effectively build out our 3x3 matrix we do
square = [board[x][y] for x in range(0, 3) for y in range(0 + 3)]

## Handling the chunks for all matrices
What are the ranges we want to actually look at? You can early on see that it's some type of pairing
(first 3 rows, first 3 columns)
(first 3 rows, columns 3:6)
(first 3 rows, columns 6:9)

(rows 3:6, first 3 columns)
(rows 3:6, columns 3:6)
(rows 3:6, columns 6:9)

(rows 6:9 first 3 columns)
(rows 6:9 columns 3:6)
(rows 6:9 columns 6:9)

You should be able to notice the pattern. Start at 0, end at 9, jump 3
```
# must be 10 in the stop place since its exclusive [0,10)
for i in range(0,10,3):
    for j in range(0,10,3):
        square = [board[x][y] for x in range(i, i + 3) for y in range(j, j + 3)]
```
```
for i in (0,3,6):
    for j in (0,3,6):
        square = [board[x][y] for x in range(i, i + 3) for y in range(j, j + 3)]
```

# Transposing a Matrix
## Why Transpose
In some problems that don't require you to transpose, it still ends up being useful to do so. Why is this the case? Well it's really easy to iterate row by row. If you need to analyze the columns of the matrix it's easier to just transpose the matrix so that you can process things by going through it row by row which again is easier to do through code.
## zip's regular function
I think in this context zip is pretty confusing at first since the original use case I think of is making a custom tuple on the fly using multiple lists. 
```
for (url, path) in zip(download_urls, download_paths):
```

This allows us to create useful data structures out of simple lists. 
## A matrix is a list of lists
The [unpacking](https://peps.python.org/pep-3132/) operator is used here which gets us a list of tuples making it more like zip(row1, row2) in the regular use of zip. What are the elements of row1 and row2? They actually represent the columns. 

Input: board =\
[["5","3",".",".","7",".",".",".","."]\
,["6",".",".","1","9","5",".",".","."]\
,[".","9","8",".",".",".",".","6","."]\
,["8",".",".",".","6",".",".",".","3"]\
,["4",".",".","8",".","3",".",".","1"]\
,["7",".",".",".","2",".",".",".","6"]\
,[".","6",".",".",".",".","2","8","."]\
,[".",".",".","4","1","9",".",".","5"]\
,[".",".",".",".","8",".",".","7","9"]]

The first element of row1 is the first column
