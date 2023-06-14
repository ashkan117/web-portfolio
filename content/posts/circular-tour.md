+++
title= "Circular Tour"
date= 2023-06-07T10:49:21-07:00
draft= true
tags=["sliding-window"]
+++


## Introduction
I wanted to discuss this problem since it was one of those where intuitively the brute force solution was not too hard to find out but 
making it more efficient was. These ended up being one of those problems where the code to the solution and the intial way you 
interpret the problem look very different.

## Breaking down 
Sometimes when you're reading solutions or editorials to problems you'll reach some quote where it's not obvious to you but to the writer it is.
They ironically start with "clearly" or "obviously". 
> Suppose we have started from the x petrol pump. Let us calculate the amount of remaining petrol in the truck at each station we go. We just need to keep adding up the difference, (petrol at i - distance of the next pump from i) at each pump. Let us say at  petrol pump, the sum becomes less than zero. What does it mean? It means, we cannot make it to the y+1 pump because there are no sufficient fuel. Also, if we start from any other petrol pump between  and , we will definitely get stuck at y pump. (Hackerrank Editorial)

We'll take a look at what the above means and how to reach that same conclusion a little more intuitively.
Let's look at an example to break things down.

Input: gas = [5,1,2,3,4], distance = [2,3,4,5,1]
Output: 3

In this example we start off strong. We have can start with 5 units of gas and the next station is only 2 miles away. 
We have 3 units of gas left in reserve so far so good.
Now at the next stop we gain an additional unit of gas but the next stop is 3 miles away. This means that our total reserve briefly goes
to 4 units of gas but once we reach the next stop we'll be at (4 - 3) = 1 unit of gas in reserve.
Now we're at index 2's stop. We gain 2 units of gas taking us to 3 units of gas in reserve. However, the next stop is 4 miles away. 
We do not have enough gas to make it.

**In the brute force way of thinking, we just say no worries. Let's just see if we can start at index 1 and see if we can complete the tour.
The observation that we need to make is that we can skip index 1 and index 2 since that work is redundant.** 

## Investigating incomplete routes
The reasoning is very logical. Let's take a look at what it means to be an imcomplete route. By that I mean that we were able to start at some
index, make it some distance, but ultimately we are not able to complete the tour.
If we were able to start a tour and make it some distance that was because the index we started at had a positive difference between the 
amount of gas we gain and the distance to the next stop. 
In the example above, we were able to get 5 units of gas and the distance to the next place was 2. 
We know for certain that since we're able to make it to the next station that this is an eligible candidate. The contribution that this index was making was positive.
**If we know that this path is invalid because we reached a negative balance for our reserve units of gas, removing a positive contribution will only hurt 
our reserve units of gas balance even further**.

That is the key observation to understand. The starting index was not the problem, that contribution is known to be positive and therefore was only helping 
us make it to the tour.

Reserve units of gas (starting at index 0) = [3,1,-2]
If we remove the contribution from the first index that will make our reserve units look like the following.
Reserve units of gas (starting at index 0) = [-2]
If we start from the second index we can't even move to the next stop since at this point we gain only 1 unit
of gas but our next stop is 3 miles away. We need to start with at least 3 units of gas to make it there.

> The starting index of an incomplete path does not provide enough contribution to make it past the problem indicies. Since the removal 
> of a good contribution only hurts us, we can skip the new starting all the way past the problem path we just investigated.

Like I said this view is really analytical/logical. The point I want to hammer home is that the contribution made by the first index
was not enough. If we had the option of increasing the first contributuion to be even more positive, then maybe it could be but we can't 
do that. The only thing that is certain is that somewhere along the way we will reach a negative balance in that path. By skipping the 
"bad path", we're hoping that a starting index past this point will gather up a large enough contribution so that when we reach this 
problem path again that we'll have enough gas to make it past it.

Hopefully now the Hackerrank quote from the beginning is a little more clear.

## Additional realizations
The total gas available and the total distance we need to cover does not change yet picking the index matters. 
If in our tour we had 1000 units of gas and we have to cover 500 units of distance it seems like a no brainer that we'd be able to complete a tour 
yet the index we start at is not trivial. 
For example if our first 3 stations are the following 

Input: gas = [1,2,3,...], distance = [3,4,5,...]

Then although we know that we can complete the tour we can also see that we can't start at the 
first three indicies since for each index we don't get enough gas to reach the next station. 
So even though we can see that we have enough overall gas to cover the tour we need to be smart about where we start.

This leads us to see that:
1. The total sum stays the same (we can calculate how much gas we have)
2. The order matters


## Avoiding the circular 
In the brute force solution you need to make sure that if you start at an index that is not the very 
first one, that we complete a circle back to get the contributions from the beginning elements.

Input: gas = [1,2,3,4,5], distance = [3,4,5,1,2]

If we choose to start at index 3, then we need to make sure that we check that we can make it to 
to index 0, 1, and 2, since we need to make sure that we stop at every station. 

### Contributions
Many times when we're trying to go from O(n^2) to O(n), we need to be smart of how we're able to combine the overall work.
This is what we do with [sliding window](https://blog.reachsumit.com/posts/2020/10/leetcode-sliding-window/) problems. 
We're able to find a way to keep track of the progress as we iterate through the problem. Usually in these problems 
we have a single factor that we're accounting for. For example in the [minimum size subarray sum](https://leetcode.com/problems/minimum-size-subarray-sum/)
we try and keep track of the sum of the elements we've viewed so far. **What makes this problem unique is that we have two contributions that we're 
keeping track of.**

> In this problem our contribution is the difference between the gas that we're receiving from this station and the cost of the distance to the next station.

### Transforming the problem
The final step we need to be able to tackle this problem in a linear fashion is converting our view of this problem as an array of penalties that we must overcome.
```python
Input: gas = [1,2,3,4,5], cost = [3,4,5,1,2]
penalty = [-2, -4, -6, -3, 0]
```

## Running Jump
We can begin viewing this problem as making sure that wherever we start from has enough positive to outweigh the negative.
Another way to phrase this is that **the prefix of the array is the problem.**

If we can't start from index 0 or 1 or 2, like in the first example that's because there is too much of a penalty in the beginning. 
We end up looking for an index where the suffix of the array has enough of a positive value to outweigh the negative prefix.

Input: gas = [5,1,2,3,4], distance = [2,3,4,5,1] Output: 3

If we choose not to start from a particular index this is because the overall cost is negative. 
we're saying that the benefits we receive from starting from this station outweigh
the cost that we accrue from the previous indicies.

## Splitting the array
The final realization that we need to make is that we can split the array up into two pieces. A good path and a bad path. 
We can only complete the tour if the good path is good enough to compensate for the bad path.

### Example
gas = [1,2,3,4,5], distance = [3,4,5,1,2]

petrolPumps = [(1,3), (2,4), (3,5), (4,1), (5,1)]

Let's view the choices as 
```python
[(1,3) | (2,4) | (3,5) | (4,1) | (5,1)]
```
If we choose to split the array at index 0 that means we're choosing the good path to be
petrolPumps[0:1] and the other path to be petrolPumps[1:]

The good path has a difference of (1 - 3) meaning there is no way to reach the next station.
We can already see that the contribution from the good path is not going to cut it. If our other path has 
a negative contribution then the tour is impossible.

The next choice is to start by picking the second partition.
petrolPumps[0:2] and other path petrolPumps[2:]
goodPath has a contribution of (1 - 3) + (2 - 4) which is -4. This is also a path.

But note that we're beginning to see how the subproblem can carry over. Even though we start from a different
index we can use the previous contribution to calculate our own contribution.


good path = petrolPumps[0:3] , other path petrolPumps[3:]
good path contribution = (1 - 3) + (2 - 4) + (3 - 5) = -6 

good path = petrolPumps[0:4] , other path petrolPumps[4:]
good path contribution = -6 + (4 - 1) = -3 

This case led us to an improvement, although since our contribution from the good path is still negative we 
won't be able to complete the tour, if the following contributions are good enough we can see that it could be.

good path = petrolPumps[0:5] , other path petrolPumps[5:]
good path contribution = -3 + (5 - 2) = 0, other path contribution = 0 (empty list)


## Additional Thoughts
We were able to travel some distance before we realize this start point is not going to work. Let's make some observations about what that implies.
One of the things I was thinking of, was to see if we can make any claim about where things went wrong? Essentially is there a specific problem 
index that causes us to fail? Maybe if we're able to skip that index maybe we can gain lots of gas on the way and handle that index last in the hopes 
of all the other indicies are helping us complete our tour.

On our route there could be a couple ways that could lead us to an invalid starting point
1. We could have many indicies after the starting point that slowly decrement our reserve gas until we are out.
2. All the indicies after the starting point are positive but the final index is overwhelming negative causing us to run out.


1. Order of contributions changes outcome
2. Total sum doesn't change
3. Inversion of choice compared to word break
4. Cost carrying over allows us to split up the array into 2 without Loss of information
5. The word done in going around the tour is the same as accepting the cost of the bad prefix
6. We only start if on an index we have a positive contribution
Logic to know we can skip bad path (starting index)
Running start idea
What determines success
Logic: The first part can only contain penalty if we don't start on that index
