+++ 
draft = false
date = 2023-03-02T17:30:57-08:00
title = "Stable Sort = Complex Sort Logic (Draft)"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

# Introduction
Sometimes you learn the hard way in life. Con is that there was a potential cost. On the plus side you will likely not forget it. 
In today's topic is brought to you by me taking twice as long to complete my interview assessment and potentially failing it for that reason.
So the questions asked you to program some logic but the order it required you to do it was the challenge. 

Note: Sort the array descending by transaction count, then ascending alphabetically by item name for items with matching transaction counts.
Example: transactions = ['notebook', 'notebook', 'mouse', 'keyboard', 'mouse']
The return array, sorted as required, is ['mouse 2', 'notebook 2', 'keyboard 1'].

I had a dictionary of frequencies and was not able to satisfy this sort requirement. Although I have done some nontrivial sorts in Python before I clearly didn't understand them well enough since I was thrown off.

# Solving this with sorted built in and its key argument

Python sorts are guaranteed to be [stable](https://en.wikipedia.org/wiki/Sorting_algorithm#Stability). "That means that when multiple records have the same key, their original order is preserved."

This seems pointless to know but in fact understanding this is crucial to satisfying the sort requirement of the question. I was able to create a dictionary of the frequencies but had to get it sorted properly.

```json
{'notebook': 2, 'mouse': 3, 'bin': 1, 'can': 3, 'button': 2}
```


So if we sort the counts by descending order we get, 
> [mouse 3, can 3, notebook 2, button 2, bin 1]

Now if we sort by ascending order for the strings we'd get 
> [can 3, mouse 3, button 2, notebook 2, bin 1]

Does this work out simply like this in code?

sorted(transaction_freq.items(), key=lambda x: x[1])

Since we're sorting a dictionary and we want to access both the key and value we'll use dict.items() to get them. We'll create a lambda function to add more directions to the sort.
You can think of the x variable as a list form of the tuple so x[0] represents the action and x[1] the value.
Python by default sorts things in ascending order but we wanted the count in descending order so we can negate the count with -x[1]

It would get two of the items then ask see if we gave it a more specific key which we did. It will get that key and compare two elements at a time.
('notebook', 2) vs ('bin', 1)
It sees that our key was the second element of the tuple which narrows the sort to 
(2) vs ( 1)
It will place whichever one is less than first resulting in 
[('bin', 1) ,('notebook', 2)]

Notice that the list is in ascending order of counts so to fix this we can just treat the numbers as negative
sorted(transaction_freq.items(), key=lambda x: -x[1])

This causes it to compare
(-2) vs ( -1) and since -2 is now smaller than we place 
('notebook', 2) then ('bin', 1)

We ultimately end up with 
> [('mouse', 3), ('can', 3), ('notebook', 2), ('button', 2), ('bin', 1)]

> We do the same thing with heaps in Python since they are implemented as Min Heaps. We can negate the values and they perform like max heaps

Python performs its sort by continuously comparing the elements until it is in sorted order. It will check is this element less than some other element.
>"The sort routines use < when making comparisons between two objects. So, it is easy to add a standard sort order to a class by defining an __lt__() method:"

## Now sort by ascending action 

Take the sorted list that we currently have and sort the actions in ascending order
sorted(sorted_m, key=lambda x: x[0])
> Output: [('bin', 1), ('button', 2), ('can', 3), ('mouse', 3), ('notebook', 2)]

What that just messed things up completely! Now we have things seemingly flipped. Well Python did what you told it to do. We told it to just sort based off the string now.
('bin', 1) vs ('mouse', 3)

bin comes earlier in the dictionary so it will place ('bin', 1) then ('mouse', 3).
It isn't obvious but notice now though that in case of ties that they are in the right order. 
can comes before mouse since they both have same count. Button comes before notebook since they also are tied with a count of 2.

That is the nature of a stable sort. 

**Define what you want to happen in the tie first, then do the sort**

TODO REST





# Resources
https://docs.python.org/3/howto/sorting.html
https://en.wikipedia.org/wiki/Sorting_algorithm#Stability
