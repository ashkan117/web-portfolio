+++ 
title= "Word Break = Choosing Partitions"
math= true
date= 2023-05-24T17:30:34-07:00
draft= false
description = ""
slug = ""
authors = []
categories = []
externalLink = ""
tags = ["O(2^N)"]
series = ["Word Break"]
+++

## Intro
Given a string of length n, there are n + 1 ways to split it into two parts.

The goal of this post is to 
1. explain why that is relevant to the brute force solution
2. explain why it's true

## Partitions
### Typical Combination and Permutations
In some problems you're making a selection from a set. To put it more formally you are creating a subset of a set. Think of problems like house robber or combination sum.
We're creating a list by choosing from the original set of items but we always have the option of not choosing an item.

> Given an array of distinct integers candidates and a target integer target, return a list of all unique combinations of candidates where the chosen numbers sum to target. You may return the combinations in any order.

Input: candidates = [2,3,6,7], target = 7
The output to this problem is [[2,2,3],[7]]
That is because we **choose** to pick those elements since they sum to the target which is 7 in this case. What makes word-break a little different is
the fact that we're forced to pick all the elements and they must be chosen in a specific order. If we make combination sum more similar to word break
it would be to split up the candidates array in such a way where each subarray has the same goal target.

Input: candidates = [2,3,5], target = 5
We can split this up into [2,3] and [5]. Notice that the following combinations don't work.[3,2] and [5] doesn't work since the original array is [2,3,5]. You can't just change the order. Hopefully this 
hints at how word-break is similar to these problems but not the same.


### Partitioning (n + 1 ways to partition)
So it is not the same as choosing elements from a list what is it like?
We are breaking up a word into pieces which implicitly means that we must include all characters of 
that word in a specific order. So the problem in this case is not which elements of the set do we choose but rather how can we break up a word.

Let's take the word "aaab", we can view breaking up this word as partitioning which is a little weird at first. 
You could cut the word in half, ["aa", "ab"], this means we're choosing a single partition. 
You could choose "a" then the remaining part of the word would be "aab". This still is choosing a partition.
Let's take at one last example. Let's imagine that in our word dictionary we have {"a", "b"}, then this means 
we'd want to split up the word into "a", "a", "a", "b". 
If we imagine adding partitions in between each character things might be more clear on how the pattern works.

a | a | a | b

So in the previous case, the correct solution required us to choose 3 partitions which resulted in breaking up
the word into 4 pieces.

### How many partitions does a string have
The only cases that our previous case missed was the empty string cases. We can partition the string "aaab" in the following way.
> | a | a | a | b | # a string of size n, has n + 1 partitions

Notice that the partitions are all different locations so they are all considered unique partitions. However in this specific problem
the result of the first partition and last partition result in the same word break. That is because if we choose the very first partition and 
only that partition our splits become "" and "aaab". Likewise, if we choose the very last partition we'll end up with "aaab" and "". 
So although we stated that "Given a string of length n, there are n + 1 ways to split it into two parts"
**So in the context of this element we can really split it up into N ways where N is the length of the string.**

## Choosing a partition = Splitting word into two
So far we've answered where the n + 1 comes from however, can we break down what the editorial meant when it says to split it into two parts? 
Well I think they meant something else there are not n + 1 ways to split it into two parts but rather there are n ways to split a word.
Each time you select any partition, you're effectively splitting the word into two, we'll take a look at this below. 
Where does the two come from then? It refers to the fact that this problem is choosing which partition leads us to a valid word split.

### Understanding how the problem can break down
Let's take a detailed look at the following example.
If the input: s = "leetcode", wordDict = ["leet","code"].
We can split up leetcode into the following ways
l | e | e | t | c | o | d | e |
We can choose anything from 1 way to partition/cut it up till 8 ways. 8 ways since the len("leetcode") is 8 and we can't have 
more cuts than there exists objects.
How many ways can we choose 1 partition?

"l", "eetcode"\
"le", "etcode"\
"lee", "tcode"\
"leet", "code"\
"leetc", "ode"\
"leetco", "de"\
"leetcod", "e"\
"leetcode", ""

In this case we see that cutting the solution into "leet" and "code" is a valid way to word break. Let's change the wordDict 
so that 1 partition is not enough
If the input: s = "leetcode", wordDict = ["l", "ee", "t" ,"code"].
We see that choosing the partition that cuts leetcode to isolate the l is what we want.
"l", "eetcode"
Now the question becomes can we split up "eetcode". Let's repeat our logic. We find the first part that matches and worry about the rest later.
Now the subproblem becomes "eetcode", how many ways can we partition this?

"e" "etcode"\
"ee" "tcode"\
"eet" "code"\
"eetc" "ode"\
"eetco" "de"\
"eetcod" "e"\
"eetcode" ""

We see that "ee" is in our wordDict, let's follow that path and now the subproblem is "tcode"
"t" "code"

This follows what we want, subproblem now is "code"
"c" "ode"
"co" "de"
"cod" "e"
"code" ""

Finally with "code" matching our new subproblem is the empty string which means we were able to successfully cut the word into 
valid pieces such that it was found in wordDict.

### Choosing partitions
We can view the problem we looked at as choosing partitions.

l | e | e | t | c | o | d | e |\
l **|** e | e **|** t **|** c | o | d | e **|**

The solution became picking 4 partitions in a specific order.


### How many ways can we choose a partition
[This](https://leetcode.com/problems/word-break/solutions/43812/o-2-n-or-o-n/comments/43066) is where we can see more formally why this problem has its roots in combinations.

a|a|a|b|

A string of length 4 has 4 places we can cut things up. 
Think of it as you go to the first area you could cut and ask yourself should I cut here? So if the characters to the left of the cut 
are in the wordDict that means that this is a possible solution. If it is not inside the wordDict it's a waste of time to entertain that option.

Only cutting the word in one location will split that word into two words. Maybe that is a valid answer like in "leetcode" and wordDict = ["leet", "code"]. 
So how many ways are there to choose one option when 4 choices exist? Well if that does not work maybe we need to choose two partition? 
We have to try all possible combinations of choosing these splits which leads us to.

$$ PowerSet(possiblePartitions) = {{4} \choose {1}} + {{4} \choose {2}} + {{4} \choose {3}} + {{4} \choose {4}} $$

We won't discuss things in this article, however since this way of solving requires the power set then we also know the time complexity. It becomes O(2^N) because in this case we are choosing the number of partitions we're taking and there are N partitions where N = len(string). Furthermore, it's because you always have the chose to choose a partition or to not choose a partition. This pattern ends up being O(2^N). If you are interested in a deeper analysis on power sets and combinations please take a look at this [great article](https://medium.com/outco/how-to-solve-power-set-c8ef7d1382ee) from Sergey Piterman. 

$$ Time Complexity = O(2^N), Space Complexity: O(N) $$

## Outro
Hopefully this posts gives you a more intuitive insight on how this problem can be viewed through the lens of parititons and combinations. I belive that once you see it in this light
you're over the major humps of breaking this problem. More importantly the goal is to have these 
insights carry over into other problems. Understanding how to iterate through subproblems is most of leetcode type problems.

## Resources
https://leetcode.com/problems/word-break/solutions/43812/o-2-n-or-o-n/comments/43066
https://leetcode.com/problems/word-break/solutions/169383/The-Time-Complexity-of-The-Brute-Force-Method-Should-Be-O(2n)-and-Prove-It-Below/comments/255523
https://leetcode.com/problems/word-break/solutions/169383/The-Time-Complexity-of-The-Brute-Force-Method-Should-Be-O(2n)-and-Prove-It-Below/comments/221580

