+++
title= "Directional Dependence"
date= 2023-06-16T08:57:43-07:00
draft= false
tags = ["patterns"]
+++


## Intro
While going over the [Flatland Space Station](https://www.hackerrank.com/challenges/flatland-space-stations/problem) problem it made me stumble to a 
pattern I have noticed before and wanted to talk more about it. In this problem 
you run into a situation where the decision you want to make is dependent on two choices. Intuitively these decisions are not 
hard but programming it could lead you into a few mistakes. In this post I want to discuss the nature of code and order.

Note: We're solving this in a multiple pass way. There is an existing solution that relies on a more logical approach. 
We're just following the more intuitive solution.

## Chicken and the egg
### Viewing things like a human
Let's take a look at the following problem.
n = 5, c = [0,4]\
[**1**,2,3,4,**5**]

This means that we have a station at the leftmost city and one at the rightmost city.
Intuitively the way we solve this problem is calculating the distances first. Any city that
has a station is 0 distance away, we can treat this as our base case.\
[0,x,x,x,0]
As humans we can see that the cities right next to space stations are a distance of 1km away.
We can fill in that information as we go.\
[0,1,x,1,0]

Finally with the last city we are equally distant from both stations so we can choose to go either way.\
[0,1,2,1,0]

### Reading it as code
Intuitively the approach we made makes a lot of sense yet there is a fundamental difference in 
the way we're able to solve this problem compared to a program. **Most importantly is way we iterate.**

As a human they way we traverse something like a list or graph is a lot more free. We can easily make any jumps we want.
What I mean by this is when we we're looking at [0,x,x,x,0], we first looked at the element to the right of the first 0, 
then the element to the left of the second 0. This simple reasoning makes a [foreach loop](https://en.wikipedia.org/wiki/Foreach_loop) 
out of the question since we're not iterating from [0,len(arr)). This is kind of like the chicken and the egg problem. Let's say our 
state of the problem looks like the array below and we're just going to iterate the array with a simple foreach loop.\
[0,x,x,x,0]

When we're at index 1, the decision we want to make is two fold. 
1. Calculate the distance to the nearest station to our left
2. Calculate the distance to the nearest station to our right

Although the decision seems straight forward we reach a catch-22. Our intention with this for loop is 
to initialize our distances, yet we require information from the "future". Our step 2 has not been done yet.
With a simple foreach loop you start at index 0 and work your way up, so you have enough information to 
calculate the nearest distance to a space station to our left but not to our right.

## Directional Dependence
This leads us to our discussion of subproblem dependence. To solve the problem at index 1 through code 
we get stuck in a situation where we rely on a solution that we have not yet calculated. In order to get around or limitation
of not being able to freely jump around in complicated ways in lists and graphs we need to be more calculated in the way we iterate.

> In this case we can change our two fold solution that was intuitive for a human but harder to code into one that is easier to code
> by removing the dependence of unsolved subproblems. 

First we'll split up our information to store the nearest distance to a station to our left and to our right in seperate arrays.
```python
# Initial distances [0,x,x,x,0]
min_distance_left = [0] * n
for i in range(1, n):
    if not_station(i):
        min_distance_left = min_distance_left[i - 1] + 1
# [0,1,2,3,0]
# Initial distances [0,x,x,x,0]
min_distance_right = [0] * n
for i in range(n - 1):
    if not_station(i):
        min_distance_right = min_distance_right[i + 1] + 1
# [0,3,2,1,0]
```

The combined information now represents what we intuitively wanted. We wanted to be able to 
know the minimum distance to a station to our left and right.

```python
final_distances = [0] * n
for (left_distance, right_distance) in zip(min_distance_left, min_distance_right):
    final_distances = min(left_distance, right_distance)
```

## Summary
Subproblems have an implicit dependence on direction when we're solving problems related to paths. We need to 
be careful to understand that the way we iterate through them is important. If we require information from a subproblem 
that we have not yet solved we need to augment the order of how we calculate things to remove that dependence.

Another way to phrase this is that we simplify the problem by gathering all the relevant information we require. Then as the 
last step we utilize the information we gained to come to a conclusion. In the problem above the steps were 
1. Solve leftmost
2. Solve rightmost
3. Use the information gained from the first two steps to generate desired information
4. Come to a conclusion based of informatin we originally wanted


## Similar Problems
In the [01 Matrix](https://leetcode.com/problems/01-matrix/) we're looking at a matrix and intuitively we'd like to look at the elements 
above us, below us, to the right, and to the left. Due to the limitations of iterating we need to break the dependence down to ensure 
we solved the subproblems we want to take a look at before we solve it. I wrote a [rough blog post](http://ashkanfaghihi.dev/posts/01-matrix-bfs/) 
about how we go about this.


