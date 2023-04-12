---
title: "Countdown Pattern"
date: 2023-03-23T15:50:13-07:00
draft: false
---


# Introduction
This pattern is pretty rare but it's easy to spot once you recognize it. Let's take a look at two problems that have this pattern. It does not improve your time or space complexity but it makes the overall approach to the code a litte simpler.


# Koko Eating Bananas
https://leetcode.com/problems/koko-eating-bananas/description/

Let's understand the problem, you're given a list of numbers that represent bananas in a pile. And we have a variable that represents hours.
Input: piles = [3,6,7,11], h = 8

This means in the first pile we have 3 bananas, 6 in the next, so on and so forth. The question wants us to discover what is the minimum of bananas we can eat each hour. Howver, there are some limitations. 
Let's say we can eat 4 bananas per hour and we're looking at the list above. **We are limited to one pile every hour**. This means although we can eat more piles than the first pile has we have to stay there until the hour is over.
So in the first hour we're able to get through the first pile, the second hour we start at the second pile but we cannot finish things. We have to wait till the next hour to try again. In the third hour we can finish the remaining 2 bananas
but as the limitation mentions we can't move to the next pile yet. Let's take a look at how our pile is changing with time.

Hour 1: piles = [0,6,7,11]\
Hour 2: piles = [0,2,7,11]\
Hour 3: piles = [0,0,7,11]\
Hour 4: piles = [0,0,3,11]\
Hour 5: piles = [0,0,0,11]\
Hour 6: piles = [0,0,0,7]\
Hour 7: piles = [0,0,0,3]\
Hour 8: piles = [0,0,0,0]

The question is really asking can we pick some number k which represents the amount of bananas we can eat per hour with the added constraint of if you eat more than one pile has in the hour you cannot
move to the next pile. In this question 4 ends up being the minimum number.

# Countdown Explored
As we can see there is a countdown of some sort. Each hour we're changing the piles list to affect the changing state. This pattern that we're looking into does not make the complexity any better but I think it makes the 
code you right a little smimplier.

Why is the countdown way where you iterate on the hour so bad? It gets messy since if you want to alter the piles list then you need to copy the data to another list so that we mutate so that we can always have the original copy. We also need to keep track
of the index since it'll only move to the next one once the bananas are done.
```python
copy_piles = list(piles)
index = 0
for _ in range(hours):
    copy_piles[index] -= k # the amount we consume each hour

    # if we've eaten all the bananas move to the next pile for the following hour
    if copy_piles <= 0:
        index += 1
```

What's the alternative to this approach? Don't think about it in terms of hours! You can calculate how many hours it would take from the information given. If a pile has 7 bananas and you are going through them with a k of 2. That will take you 
7 / 2 = 3.5 hours. Do we round down or up? Well remember the weird rule that you can't start until the next hour after you finish the pile. So even though we're 3.5 hours in we have to wait until the 4th hour to start the next pile. This 
leads us to the following equation.
> time = ceil(bananas_in_pile / k (rate of bananas we're eating))

The ceil function just makes sure we round up in the case of decimals as we've seen.

# Conclusion (Avoid Countdown)
If you have some problem where it feels like this countdown since we're dependant on some form of time or some rate of change think about using this pattern. It's not a pattern if it's a one off event so feel free to take a look at [Car Fleet](https://leetcode.com/problems/car-fleet/description/) which is another problem where I have noticed this pattern. It seems like an unnecessary way to approach the problem but doing the simple countdown way led me to bugs in both of these problems.
