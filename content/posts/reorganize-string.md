---
title: "Reorganize String"
date: 2023-03-16T17:34:25-07:00
draft: true
---


# Introduction
The patterns that you run into for this problem are pretty interesting which makes it slightly challenging to understand how to iterate through when you code up the solution.
It's a placement problem.

# Transforming the input
In this case the input is not actually helpful and if anything it serves as a detriment. Let's take a look at if the input was "apple". The given input does not matter! This is really a placement type problem. Given a list of candidates how do you place them?
This makes it easier to see that the more important aspect is the frequencies of the characters.

# Understanding the pattern

# Round Robin Fails
{a: 3, b: 2, c: 2}
a b c a b c a

{a: 3, b: 1, c: 1}
a b c a a # fails

a b a c a 

# How frequencies matter for our placement
# Most Frequent Character is the problem
One thing to notice is that if we always have the same frequency for all character the problem is easy and we could get away with round robin. If we don't have equal characters you can see that the one with the most characters
is the problem character.
As an example, take {a: 2, b: 2, c: 2}. Round robin would work fine in this case but let's change things slightly and see what happens. {a: 5, b: 2, c: 2}. Now since *a* has more elements the 
problem becomes *can we space things out enough* in order to prevent placing adjacent characters.
a b a b a c a c a

This shift get's us close to the right approach. We see that the element with the most characters is the trouble element. We want to place that one first then add a buffer character. This also explains why the round robin fails.
{a: 3, b: 1, c: 1}
a b 

After we place *b* remember that the problem character is *a*. We want to save all other characters in order to help us space things out. By placing *c* right after *b* then we're wasting a character
that could have helped us space things out. 

## Problem character can change
Another question to ask yourself is should I place things in a greedy manner? For example if we have {a: 3, b: 3, c: 2} should I just focus on placing *a* and *b* first then worry about the rest?
a b a b a b c c

You can see if you place those two characters until they are done then *c* just became the problem child and we have no other characters to space things out.

## Lowest frequencies characters are high value
One thing that you can also notice is that by avoiding placing the characters with lower frequencies you're giving yourself more wiggle room. 

{a: 5, b: 3, c: 1}
If a place *c* after the *a* then I have less characters to help me space things out.
a c 
{a: 4, b: 3 }

## Limit
One thing that you can also notice is that by avoiding placing the characters with lower frequencies you're giving yourself more wiggle room. 

{a: 5, b: 3, c: 1}
a ... 
{a: 4, b: 3, c: 1}

a b ... 
{a: 4, b: 2, c: 1}

len(trouble) = len(rest) + 1

Why






This idea of spacing things out intuitively means we 
1. Don't want to go round robin since it doesn't save the characters for the trouble character
2. The characters with the lowest frequencies are high value. Meaning you don't want to place them for as long as you can

# Grab Most Frequent with a Hold
This is where I could not connect things properly since the pattern gets confusing. We see that there is a problem character that is dependent on frequency however h



{a: 1, b: 5, c: 3}
Let's say we start with b
b 

What should we do next? It might not be clear what we should do but we know something that we should not do. We should not place b again.
bb # invalid 

So the same question is a little more clear, should we place *a* first or *c* first? Does that even matter? Let's place *a* first.
ba 
{a: 0, b: 4, c: 3}

There is seemingly no issues since we can place the rest of the elements just fine. 
babcbcbcbc

Raising the count of *b* by a single element is all it takes to break things when we place *a* first.
{a: 1, b: 6, c: 3}
babcbcbcbc
{a: 0, b: 2, c: 0}

It also might not be obvious but this could be solved if we had picked *c* first to place.



