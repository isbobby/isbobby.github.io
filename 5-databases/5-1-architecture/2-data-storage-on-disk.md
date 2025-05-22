---
title: Data Storage on Disk (WIP)
parent: Database Architecture
nav_order: 2
---
1. Refer to previous chapter of [Storage Media](https://isbobby.github.io/5-databases/5-1-architecture/1-physical-storage-media.html), one constraint that guides the following design is that disk IO is much, much slower than CPU, our system should therefore aim to minimise such IO operations
2. We again start with an assumption, where everything is memory and addressing is cheap. How would we design a data structure that's optimal for search by different keys, search by range, insert & delete.

## Initial Candidates
First, we need to identify what we want to store - to simplify the following design, we assume our data comes in the form of key to tuple format, where `key : (a,b,c)`.

We want our database to have performant query by key(s), and query by range. By this requirement alone, we can eliminate a naive unsorted array or linked list data structure, as they both have $O(N)$ look up time.

Perhaps keeping the array in a sorted order works, but if we perform insert and delete, we may end up doing $O(N)$ operation. It's also hard to allow concurrent access when there are ongoing writes - even if they are at a different part of the array.

When look up is involved, we naturally think about hash map and balanced binary search tree.

Although hash map provides $O(1)$ look up time, it will not be able to execute range query very well, the complexity of range query deteriorates to $O(k)$ where $k$ is the size of the range interval.

Whereas a balanced binary search tree would be able to provide $O(logN)$ look up, and $O(logN)$ range look up, where `N` is the total number of keys.

Insert and delete for both data structures are not a big issue.
## Moving to Disk
Balanced binary search tree has slower look up, but it provides a more consistent performance for all operation. We should explore in this direction.

