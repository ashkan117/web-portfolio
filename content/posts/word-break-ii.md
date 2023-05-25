---
title: "Word Break Ii"
date: 2023-05-25T13:31:49-07:00
draft: true
---

## Intro
With recursion it's all about the recursive leap of faith and thinking in a lazy manner. It's really hard to solve the problem as a whole however the 
subproblems are not too bad to solve. 
> Input: s = "applepenapple", wordDict = ["apple","pen"]


It's not actually to hard to imagine that we can code something where we eventually realize that "apple" is part of s, then 
in order to see if the full string can be segmented we'd just need to determine if "penapple" can also be segmented. The first part is not too bad 
so we'll take care of that work, **trust** that the recursion can handle the rest.

If we eventually reach the case where our string is empty that means that we were able to continuously chop off the first part of the string. If we 
ran out of characters it means everything was successfully found in the wordDict.
