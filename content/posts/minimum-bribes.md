---
title: "Minimum Bribes"
date: 2023-06-06T10:20:45-07:00
draft: true
---


## Introduction
When I was first solving this problem I was able to pass some of the test cases but was caught off guard when I would fail the following case.

[1,2,5,3,7,8,6,4]

This case is a valid case (it's not considered chaotic) however you'll notice that many bribes occurred in an a non trivial way.

## Naive first approach (Counting people who jumped forward)
Once we notice someone out of place it must be someone who bribed another person. This is because bribers, move forward in the list.
In the example above the first person we notice is person 5 since we jumped ahead of where he should be. Once we notice this person, 
see where this person should be and consider all the jumps he made as bribes. So if he moved forward two spots in the array he must
have bribed two people. 

- Person 5 should be at index 4 however we are currently at index 2. This means this person has bribed two people. 
- Person 7 is the next person we notice jumped ahead. He should be in index 6 yet he is in index 4. This means he bribed two people
- Person 8 is the next person we see that jumped ahead. He should be in in index 7 yet he is in index 5. He also bribed two people.

My line of reasoning at this moment is:
1. You can tell if a person is out of place since their value determines where they should have been
2. A briber moves forward in the list
3. A person who has been bribed has moved backwards

So far all our observations are correct however we are missing one case. Person 6 has also bribed person 4 yet with our 
viewpoint we cannot tell since he ultimately moved backwards in the list. 

> Once you found a solution that works make sure you test it on a non trivial case. Many actual assessments hide these difficult cases 
> from you meaning it's an important skill to think of them independently. These cases are far more important since they lead to 
> major realizations

**These are not swaps**

## Why it falls short (Nested Decisions)
The reason I went down the path that I did was because you notice bribers first. Since we notice them earlier this led me to viewing an answer with them 
as the focus. Let's see why my approach isn't good enough. The first briber you'll notice if you iterate through the list from the start is person 5. 
You can see that person 5 is not where it should be and we know that it is the briber not the bribed. We know that since it moved 
up in the line, however there is one thing we cannot know from this viewpoint **how many people did this person bribe.** This means we're unable
to answer the question of how many bribes occured in non trivial cases.

I will be writing a [post]({{< ref "abandoning-faulty-logic.md" >}}) about it later but one way you can recognize that your current way of thinking 
is just too difficult to reach a conclusion in a complex example. In my case, my logic actually worked for simple cases, however if we take a look at the example above
it's actually really difficult to determine the order of things with the current way of thinking. Don't believe me? Try it out! It will end up leading to lots 
of trial and error.

The main case that I failed to consider is that "nested decisions" will have cases where the briber ends up moving backwards in the list. 
Let's start with the following line 
```python
[1,2,3,4,5,6]
# Let's choose person 4 to bribe person 3
[1,2,4,3,5,6]
# let's choose 5 to bribe person 4. 
[1,2,5,3,4,6]
```
All of a sudden person 4 which we know made a bribe fell backward in the list. **This simple case is not too hard to think of yet 
it ensures that my second observation of the problem was wrong!**

## Breaking down the problem further
We're close to solving the problem however we need to alter our solution to be able to count the number of bribes 
in more complex configurations like the one above. In that example we can see that 5 bribed someone but if we look 
at where 5 "should be" then we see something more complicated has happened. This lead me to consider how the 
problem of counting the number of people who have been bribed is not the same as counting the number of people 
who bribe others. In the case above the list of people who bribe are [5,6,7,8] however the people who were bribed are [3,4,6]. 
Moreover, 
1. Person 5 bribed two people (3,4)
2. Person 6 bribed one person (4)
3. Person 7 bribed two people (4,6)
4. Person 8 bribed two people (4,6)

The point I am trying to get at is that this problem is more difficult than it seems. If someone asked you to 
take the problem [1,2,5,3,7,8,6,4] and come up with the order of bribes it's actually pretty hard.

## Switching Perspectives
This complex case becomes easier to if you view it as counting the number of people 

```python
[1,2,3,4,5,6]
# Let's choose person 4 to bribe person 3
[1,2,4,3,5,6]
# let's choose 5 to bribe person 4. 
[1,2,5,3,4,6]
```

From 3's perspective 


```python
[1,2,5,3,7,8,6,4]
# swap person 3 with person 5
[1,2,3,5,7,8,6,4]
# swap person 6 with person 8 and 7
[1,2,3,5,7,6,8,4]
[1,2,3,5,6,7,8,4]
# swap person 4 with person 5,6,7,8
[1,2,3,4,5,6,7,8]
```

## Resources
https://coding-gym.org/challenges/new-year-chaos/
https://csanim.com/tutorials/hackerrank-solution-new-year-chaos
