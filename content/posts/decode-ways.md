+++ 
draft = false
date = 2023-01-30T09:09:58-08:00
title = "Leetcode 91: Decode Ways (Bounding your recursive calls)"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

# Why I like this problem
https://leetcode.com/problems/decode-ways/\
A lot of the canonical recursive problems you start out with have really simple code. Things that come to mind are https://leetcode.com/problems/fibonacci-number/,  https://leetcode.com/problems/climbing-stairs/ and https://leetcode.com/problems/house-robber/. This is because the recursive calls are really simple, it doesn't really require you to understand how the recursive calls unwind. Decode ways does this will for a few reasons.
1. The subproblems are not trivial to "build"
2. You can't blindly make recursive calls

Note: Most solutions end up doing a bottom up approach which is better, I just want to discuess the top down approach since that is usually my first way to tackle these problems.
This also reminds me of **Generate Parentheses** since we have some bounds on when we can make recursive calls.


# The subproblems
There were a few things that you must recognize before you can tackle the solution. One of them isn't too difficult to notice. The easy case with recursion is usually just take one off the current input. If we have "226" we can see that the solution does depend on "22". "22" therefore depends on "2". The case that's a little harder to notice is that you always want to consider two character substrings as well. Given "226", the solution also depends on what the results of "26" subproblem are. Likewise when we're looking at "22" we need to treat this as a two digit number as well. This leads to the 3 paths for input "226".

Input: s = "226"
Output: 3
Explanation: "226" could be decoded as "BZ" (2 26), "VF" (22 6), or "BBF" (2 2 6).

The cases aren't exactly the most complicated but I did run into issues trying to code this breakdown of subproblems with off by one errors. 

# You can't blindly make recursive calls
In the simple recursive problems I mentioned above you can simply return based off rec(n - 1) + rec(n - 2). You can always break the problem down in simple ways like this. In Decode Ways this is not the case. You need to handle the fact that the "Decode" we're doing is only valid from "1" - "26". Let's look at the case "336" to explore this a little more. We can take "6" and we're left with "33". This is not a valid way to decode things though since it's not a valid numerical representation for a character. Therefore we "prune" this path, we don't want to continue down recursively since it's not fruitful.

Another case is "220". You can't always assume the single character way works as well. You can't break things down into take "0" then solve the subproblem of "22" since "0" is not valid. This means we need another check for "0".

This leads us to our two important checks, 
1. You can only take a single digit character if it is not "0"
2. You can only take a two digit character if the value it represents is [10, 26] inclusive. 



