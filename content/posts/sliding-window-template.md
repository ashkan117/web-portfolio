---
title: "Sliding Window Template"
date: 2025-02-23T15:53:10Z
draft: true
---

## General Resources

- [Great Visual Resource](https://www.hellointerview.com/learn/code/sliding-window/variable-length#the-sliding-window)
- [Detailed breakdown](https://faangshui.com/books/guides/chapters/core-concepts/lessons/sliding-windows)
- [Good visual walkthrough of problem](https://www.tiktok.com/@visual.leetcode/video/7267330177376718126)

## Motivation

My motivation for this post is not to breakdown what the sliding window is,
but to help give more structure
to the concept when actually implementing it. I've found many times
that many times I'd be able to reason through how we could use the sliding window
but I would constantly trip up over the details of how to actually implement it.

The errors I'd face were

- Checking the window for an answer, but at the wrong time
- Incorrectly shifting the window to grow or shrink too early or too late

## Fixed Length

Let's start with the slightly easier version to visualize, which
is the fixed length sliding window.

### General Template

```python
def fixed_length_sliding_window(nums, desired_window_size):
    state = 0  # choose appropriate data structure
    start = 0
    max_ = 0

    for end in range(len(nums)):
        # extend window
        state += nums[end]  # add nums[end] to state in O(1) in time

        if end - start + 1 == desired_window_size:
            # INVARIANT: size of the window is k here.
            max_ = max(max_, state)

            # contract window
            state -= nums[start]  # remove nums[start] from state in O(1) in time
            start += 1

    return max_
```

### end - start + 1 means the current window size

Why the plus 1 in `end - start + 1`? Well it has to do with the
fact that the `end` and `start` are represent the indices of the
sliding window, and the `end` index is exclusive!

That just means if we have some string "inclusive" and we want
we say we randomly take a look at the substring where the start is
at index 2 and the end is at index 5, then let's see what that results in.

"clu"

Now when working with sliding window problems, we need the size
of the current window. One way we could do that is `len(substring)` but
that is a bit inefficient, since intuitively we're recalculating the length.

Instead, we can use the indicies that we have to calculate it
using simple math. Let's go back to the "clue" example. We
got this substring from the index 2 to index 5,
and if we wanted to get the length of the substring, we could do
`end - start + 1`, which is `5 - 2 + 1` which is `4`.

### Order of Growing, Shrinking, and Analysis

One thing that I would constantly trip up over is the order of growing,
shrinking, and when we check for some condition (like finding the maximum).

We can fix this order to be

1. Grow the window
2. If we're at the desired window size, check for some condition
3. If we're at the desired window size, shrink the window

Note that in this case we'd only need to shrink the window if we
were at the desired window size. This is because we'll always start
the loop by growing the window and we always want to make sure
that we stay fixed at our desired window size.

## Dynamic/Variable Length Sliding

### General Template

```python
def variable_length_sliding_window(nums):
    state = # choose appropriate data structure
    start = 0
    max_ = 0
    for end in range(len(nums)):
        # extend window
        # add nums[end] to state in O(1) in time
        while state is not valid:
            # repeatedly contract window until it is valid again
            # remove nums[start] from state in O(1) in time
            start += 1

       # INVARIANT: state of current window is valid here.
       max_ = max(max_, end - start + 1)

    return max_
```

### Order of Growing, Shrinking, and Analysis

1. Grow the window
2. Update the window to be valid again (aka shrink if needed)
3. Check our desired condition now that the window is valid

## Fixed Length Problem: Permutation String

An application of this would be the following [leetcode problem](https://neetcode.io/problems/permutation-string).
I found this solution from [niits](https://leetcode.com/problems/permutation-in-string/solutions/6260852/video-sliding-window-hashmap-solution/)
intuitive to me.

### Code

```python
class Solution:
    def checkInclusion(self, s1: str, s2: str) -> bool:
        if len(s1) > len(s2):
            return False

        freq_one = defaultdict(int)
        freq_two = defaultdict(int)
        for i in range(len(s1)):
            freq_one[s1[i]] += 1
            freq_two[s2[i]] += 1
        if freq_one == freq_two:
            return True

        left = 0
        for right in range(len(s1), len(s2)):
            freq_two[s2[right]] += 1
            freq_two[s2[left]] -= 1


            if freq_two[s2[left]] == 0:
                del freq_two[s2[left]]
            left += 1

            if freq_one == freq_two:
                return True
        return False
```

### Interesting Properties: Initial work of preparing the sliding window

In some cases it's more intuitive to think about iterating through things
once the window size is already at the desired size. I want to discuss
a few interesting side affects of this approach.

We have to have two identical conditions to check for some condition. If you
notice we do the check for `freq_one == freq_two` twice. That's because

- before the loop starts we have the first valid window sub-problem
- when the loop starts we always grow the window size which would cause
  us to skip checking the first valid sub-problem.

### Interesting Properties: We delete the keys in the dictionaries

One other thing that can seem strange is that we remove keys once they hit
zero. Why?

Well that's because when we want to compare the dictionaries down
the line we'll run into a situation where although
logically something is "equivalent" in our eyes, technically they are not.

What I mean by that is let's say we're comparing the two dictionaries that represents
"2 a's, 1 b and no c's"

```python
{a: 2, b: 1}
{a: 2, b: 1, c: 0}
```

I think intuitively we can tell that both mean the same thing but in code it's not
if we're comparing the structure of both dictionaries, then the second one has additional
keys that the first one does not have.

## Dynamic Length: Longest Repeating Substring with Replacement

In this [leetcode propblem](https://neetcode.io/problems/longest-repeating-substring-with-replacement)
we take a look a situation where the condition for
the state of the problem depends on what the window looks like in the invalid
state.

I'll be using some visuals from [hellointerview](https://www.hellointerview.com/learn/code/sliding-window/longest-repeating-character-replacement)

### Code

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        left = 0
        freq = defaultdict(int)
        most_freq = 0
        result = 0

        for right in range(len(s)):
            char = s[right]
            freq[char] += 1
            most_freq = max(most_freq, freq[char])
            while right - left + 1 > most_freq + k:
                freq[s[left]] -= 1
                left += 1

            result = max(right - left + 1, result)

        return result
```

### Interesting Properties: We use state that spans all windows

When solving this problem one day I was slightly unsure of the following? In my
head, my intuition is that we only take snapshots of the world when the state is
valid. My reasoning was that if we keep data from a situation, where
the state is invalid, wouldn't that make the data we get invalid. Short answer,
no.

Let's take a look at this example case

```bash
s="AAABABB"
k=1
```

At some point we'll reach the case where the window becomes
`AAABAB` which is invalid since we'd like to replace the
2 Bs that we see but we were only allowed a wiggle room
of 1 error. Therefore it's important for us to keep
