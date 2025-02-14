---
title: Dynamic Programming
parent: Data Structure & Algorithms
nav_order: 1
---
# Dynamic Programming
Dynamic programming is a **mathematical optimisation**, and a **algorithmic paradigm**.

In the sense of **mathematical optimisation**, it refers to simplifying a complex decision, by breaking it down into a sequence of simpler decisions over time.

The problem must have two key attributes in order for dynamic programming to be applicable
1. **optimal substructure** - global optimal solution can be obtained by combination of optimal solutions to its subproblems
2. **overlapping subproblems** - when the same subproblems are solved again and again

If the problem only has optimal substructure, but **non overlapping subproblems**, it can be solved by [Divide and Conquer]() instead.

# Program Structure
Top down / DFS, where we start off `dfs()` with the root, recursively going down to the base cases.

```python
def fibonacci(n, memo=None):
    if memo is None:
        memo = {}
    if n <= 1:
        return n
    if n not in memo:
        # Recursive computation with memoization
        memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo)
    return memo[n]
```

Bottom up, where we start off working on the sub-problems.
``` python
def fibonacci(n):
    if n <= 1:
        return n
    # Initialize an array to store Fibonacci numbers
    dp = [0] * (n + 1)
    dp[0], dp[1] = 0, 1  # Base cases
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    return dp[n]
```
