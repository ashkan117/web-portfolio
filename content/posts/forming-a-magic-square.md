---
title: "Forming a Magic Square"
date: 2023-06-12T18:06:53-07:00
draft: true
---

## Introduction

This problem follows a trend I have noticed in hackkerank problems. The problem is very challenging if you attempt to solve it a more straightforward way.
The observations you need to make are fairly involved and obscure! Even more so in this problem since many of the solutions I have seen use the fact that 
the solution is a 3x3 matrix. Regardless, the solutions all end up doing some form of brute force check.

## What is the magic number?
This problem was difficult to understand to begin with. 

## How to find the magic number?

## How to choose the optimal number of changes

### Wrong viewpoint (Swaps is too dificult)
My original line of thinking is there should be a way where you make a smart decision 
of which choice to make. I cannot prove that there is no way to do this but I can describe 
why that choice is difficult to discover. 

With our knowledge of the magic number being easily calculatable and the the matrix values are all unique,
we can make some additional observations. For a 3x3 matrix, there are pairs of numbers that must be tied together.

DECLARE OBSERVATION that [1,25] must be in the same

There are too many variations to choose from. Although we see the pairs that must exist, those pairs can be down a column,
a row, or on the diaganal, furthermore they could happen on any row, column, or diaganol. 

### Correct viewpoint

