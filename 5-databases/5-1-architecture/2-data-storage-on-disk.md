---
title: Data Storage on Disk (WIP)
parent: Database Architecture
nav_order: 2
---
1. Refer to previous chapter of [Storage Media](https://isbobby.github.io/5-databases/5-1-architecture/1-physical-storage-media.html), one constraint that guides the following design is that disk IO is much, much slower than CPU, our system should therefore aim to minimise such IO operations
2. We again start with an assumption, where everything is memory and addressing is cheap. How would we design a data structure that's optimal for search by different keys, search by range, insert & delete.

## Three Key Considerations
1. Storage efficiency - how many bytes in the storage is required to represent one byte of data.
2. Access efficiency - how many steps are required to read data
3. Update efficiency - how many steps and how many changes are required to perform one update
## Look up mechanism
First, we need to identify what we want to store - to simplify the following design, we assume our data comes in the form of key to tuple format, where `key : (a,b,c)`.

Assume we have a data tuple which looks like `id, name, age` where `id` is the key, we can arrange them in a contiguous array
```
[1,alice,12],[2,bob,13],[3,charlie,11]
```

To have efficient look up by `id`, we can create a "header" in front of the contiguous array, serving as an index to the offset of the data. The index will be stored in a sorted format which corresponds to the ordering of data. This allows for quick look up via binary search.
```
[1,4][2,5][3,6][1,alice,12],[2,bob,13],[3,charlie,11]
```

Now, we may want to extend this efficiency to another element in the tuple, say `name`. We can follow the approach by creating new tuples of `(index,location)`, allowing efficient access to the data (spoiler - not preferred).
```
[alice,4][bob,5][charlie,6][1,4][2,5][3,6]
[1,alice,12],[2,bob,13],[3,charlie,11]
```

Now both `id` and `name` are indices allowing for fast look up, they live on the same file containing this contiguous array. They both reference the **actual offset** in the array. We can refer to these types of indices as **clustered index** - the index and data are clustered together.

Clustering index with data allows for quick look up via offset, but it increases the complexity for insert and deletes. For example, what happens if remove the `[2,bob,13]` entry? We will have to delete `[bob,5]` and `[2,5]` indices. We can mark them as null for now.
```
[alice,4][__,__][charlie,6][1,4][_,_][3,6]
[1,alice,12],[3,charlie,11]
```

We may want to insert data as well, and having multiple indexes in the same header makes inserting more complicated.
```
[alice,4][__,__][charlie,6][david,9]
[1,4][_,_][3,6][4,7]
[1,alice,12],[3,charlie,11],[4,david,9]
```

In addition, in the example above, the `name` happens to be sorted - what if the `name` is actually not sorted by `id` (like `(1,bob),(2,alice)`)? This means to in order to keep both index sorted, we will have to re-order at least one of them for every insert.

For the above reasons, it makes data operation much easier to manage when we have only one clustered index per array, and we can abstract alternate index out. Let's use the first data snippet before the writes. We make an improvement by allowing **non-clustered index**, which exist in a separate array.

```
Data Array
[1,4][2,5][3,6]
[1,alice,12],[2,bob,13],[3,charlie,11]

Additional Index Array
[alice,4][bob,5][charlie,6] // refers to the offset in data array
```

This approach seems better as it solves the ordering issue, but having additional index referencing the actual location of the data in the data array can be tricky. This is because the actual location of the data may change as data gets deleted. Hence, instead of referencing the actual location of the data element in the data array, the secondary index can simply refer the **clustered index** of the data (or primary key).
```
Data Array
[1,4][2,5][3,6]
[1,alice,12],[2,bob,13],[3,charlie,11]

Additional Index Array
[alice,1][bob,2][charlie,3] // refers to the primary key / clusterd index
```
## Initial Candidates
We want our database to have performant query by key(s), and query by range. By this requirement alone, we can eliminate a naive unsorted array or linked list data structure, as they both have $O(N)$ look up time.

Perhaps keeping the array in a sorted order works, but if we perform insert and delete, we may end up doing $O(N)$ operation. It's also hard to allow concurrent access when there are ongoing writes - even if they are at a different part of the array.

When look up is involved, we naturally think about hash map and balanced binary search tree.

Although hash map provides $O(1)$ look up time, it will not be able to execute range query very well, the complexity of range query deteriorates to $O(k)$ where $k$ is the size of the range interval.

Whereas a balanced binary search tree would be able to provide $O(logN)$ look up, and $O(logN)$ range look up, where `N` is the total number of keys.

Insert and delete for both data structures are not a big issue.
## Moving to Disk
Balanced binary search tree has slower look up, but it provides a more consistent performance for all operation. We should explore in this direction.

