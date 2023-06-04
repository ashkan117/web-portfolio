+++
title= "Longest Increasing Subsequence (Part II)"
date= 2023-05-30T12:59:31-07:00
draft= true
series= ["Recursion to Dynamic Programming"]
+++

# Intro
Stumbled into an amazing [YouTube Channel](https://www.youtube.com/playlist?list=PL4l4bxjizQAGMPbDzxGKUekxObBN5DSSO) that 
helped describe this pattern that appears in DP problems

## 1D DP Solution (Reconnecting the previous element index with the choice)

## Using the direct prefix of the problem does not work

### Multiple candidates are an issue
This problem lead me to realize that usually what we do with DP problems is to store the result 
of a subproblem that is a prefix of the current problem.

That means that when we recursively view a subproblem we say what if we solve a part of it 
and let the recursion take care of the rest. I thought that was all recursive problems but that is not always the case.

Let's take a look at the following example to see why.
nums = [4,8,1,2,4,1,9]
We'll have a DP array to store the solution to the prefix subproblem. That means that if we're at the last index, the prefix 
view is "lazy recursive choice" where you say I will solve an extremely easy problem since I am lazy and let the recursion 
do the rest of the work. In this case, assume we know the previous element chosen so that means we casubproblem. That means that if we're at the last index, the prefix 
view is "lazy recursive choice". If we are at the last index and know the previous element chosen, the work I do is not too bad. 
If my value is greater than the previously chosen element then I can just add 1 to the previous subproblem. In this view 
the previous subproblem represents the prefix which is the solution to nums[1...i] which is [4,8,1,2,4,1].
1. If our element is greater than the previously chosen element do 1 + dp[i - 1]
2. Else, this is the start of a new sequence. Just return the previous max answer dp[i] = dp[i - 1] 

The reason this falls short is because there can be cases where multiple candidates can be chosen.

When we're at index 4 of the example problem and we're looking at element 2, what is the LIS? It's a tie! 
If we want to choose index 4 (value 2) then that does lead us to a sequence of (1,2) which is tied with 
the longest increasing subsequence that we had due to (4,8). Now if we go to index 4 which is another value of 4, now [1,2,4] wins out and our LIS has a max value of 3.

However note that this breaks things since our solution depends on the previous value. In these cases where there are multiple candidates, **you cannot easily 
represent the fact that the previous value can be multiple values.**
![multiple candidates](/images/lis-multiple-candidates.jpg)

nums = [1,2,4,8,1,9]

> Having **multiple candidates** for a solution means you cannot easily define the recurrance relationship.
> Multiple candidates make it so that the optimal solution depends on having additional surrounding context.

## Removing ambiguity by including constraints
Ambiguity is not always a problem but in some areas it can be. An example of this shows up in [creating programming languages](https://en.wikibooks.org/wiki/Introduction_to_Programming_Languages/Grammars).
The solution to ambiguity is adding constraints since it makes things more specific.

### including x[i] constraint
Now let's change the definition dp[i] to mean longest increasing subsequence in x[1...i] **including x[i]**.

## Conclusion

We add a constraing to 
1. Eliminate ambiguity
2. There is more information to be gained 



## Resources
https://leetcode.com/problems/longest-increasing-subsequence/solutions/1326552/optimization-from-brute-force-to-dynamic-programming-explained/
https://www.youtube.com/watch?v=xRv-tnKY4HU

