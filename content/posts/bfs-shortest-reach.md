+++
title= "Bfs Shortest Reach"
date= 2023-06-12T17:43:27-07:00
draft= true
tags=["graphs"]
+++

## Introduction
This hackerrank problem at a high level is pretty easy since it's a straight forward BFS algorithm however the few differences in the problem statement
led me to stumble to a coding solution. 

1. The input given is not helpful, we need to transform it in a more.
2. The mismatch between the count starting at 1 and the index starting at 0 
3. The situtation where we the result returned has a size of N - 1 and the element missing depends on the starting point

## Building an adjacency matrix
The input is given as a list of numbers that represent the connections. [[1,2], [1,3], [3,4]]. The issue is that there is no way to treat
this as a traversal in this view. The first index represents the connection between 1 and 2 however if we wanted to "go down this route" there is no 
great way.

```python
graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)
# results in {1: [2,3], 2: [1], 3: [1,4], 4: [3]}
```

Now lets take a look at our new representation.
{1: [2,3], 2: [1], 3: [1,4], 4: [3]}

We know that the starting node is 1 in this case, we want to traverse all its children.

## Level Order traversal
Now the second part of the problem is to determine the "shortest reach". That is the strength of BFS algorithms. 
Howeveer there are many ways to do this approach we'll do the more general approach first then a small twists that
leverages the nature of the current problem.


```python
distances = [-1] * n
distances[s-1] = 0
queue = deque([s])
visited = set()
depth = 1
while queue:
    node = queue.popleft()
    # go through all the children
    for neighbor in graph[node]:
        if neighbor not in visited:
            distances[neighbor-1] = 6 * depth
            queue.append(neighbor)
    depth += 1
```
