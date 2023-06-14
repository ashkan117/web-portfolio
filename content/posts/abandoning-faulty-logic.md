---
title: "Abandoning Faulty Logic"
date: 2023-06-13T12:05:36-07:00
draft: true
---


## How do you abandon faulty logic?
When trying to solve a problem there are many times where your initial line of reasoning is not the correct way. In this problem
I chose to focus on counting the bribers, this approach worked for some of the test cases but not all of them. There was no 
extension I could make to this line of thinking to fix the problem since the core of my approach was faulty.

The only way you can do this is by realizing this path's shortcomings. Simply put you need to be able to recognize that this 
This ends up being like a DFS problem. If you're current path is indefinite this means you will spend your time going down 
a path that doesn't give you an answer yet the correct path is closeby if you stop checking.


1. The current path is too difficult
2. The current path is incomplete

## Too Difficult 
In solving the [minimum bribes]({{< ref "minimum-bribes.md" >}}) and [forming a magic square]({{< ref "forming-a-magic-square.md" >}}) problem we were trying to solve this 
problem the wrong way. 

1. Your approach kind of works but in complex situations it becomes too complicated. In the minimum bribes problem my solution worked in the simple cases but
not in the complex cases. In the complex cases my approach became unruly. It was too difficult to apply to complex cases.
Too much **trial and error**.

## Tips
Simple cases are not good indicators. Once you thought of a solution that works for a simple case find a more challenging
problem and apply it to that. If it fails abandon this way of thinking. (Would have helped with minimum bribes)

> Once you found a solution that works make sure you test it on a non trivial case. Many actual assessments hide these difficult cases 
> from you meaning it's an important skill to think of them independently. These cases are far more important since they lead to 
> major realizations


## Coming up with good test cases
Nested dependence. In many problems there is a choice being made. Linking choices together is a good way of coming up with nontrivial cases.
Let's look at minimum bribes.
[1,2,3,4,5,6]
Let's choose person 4 to bribe person 3. It will result in 
[1,2,4,3,5,6]
So far so good. However let's choose 5 to bribe person 4. This case is not too difficult to think of and it would have 
worked in proving my initial line of reasoning wrong very early in the process.
[1,2,5,3,4,6]

