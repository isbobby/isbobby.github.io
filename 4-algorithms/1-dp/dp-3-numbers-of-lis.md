---
title: Uncrossed Lines
parent: Dynamic Programming
nav_order: 3
---
### Number of Longest Increasing Sub-Sequence ([Link](https://leetcode.com/problems/number-of-longest-increasing-subsequence))
# Warm up with LIS
We can review the question *Longest Increasing Sub-Sequence* as a warm up.

Given an integer array, we want to find the length of the longest increasing subsequence ([link](https://leetcode.com/problems/longest-increasing-subsequence/description/)).

For example, the longest subsequence of 
```
arr = [10,9,2,5,3,7,101,18]
LIS =       ^   ^ ^  ^ (4)
```

For this problem, the optimal substructure can be defined as: given an array of length $$N$$, the longest subsequence $$dp(N)$$ will be $$Max(dp(i))+1$$ where $$i=0,1...N-1$$ **if** the element at $$i$$ is smaller than element at $$N$$. If no such element exists, then $$dp(N)=1$$ (element itself).

We can store the longest increasing subsequence up to $N$ to prevent re-computation. We can refer to the following example to see how the intermediate results are used to give the final solution.
```
arr = [10,9,2,5,3,7,101,18]
dp  = [1, _,_,_,_,_,_  ,_] // at 10
dp  = [1, 1,_,_,_,_,_  ,_] // at 9
dp  = [1, 1,1,_,_,_,_  ,_] // at 2
dp  = [1, 1,1,2,_,_,_  ,_] // at 5, 2 is smaller
dp  = [1, 1,1,2,2,_,_  ,_] // at 3, 2 is smaller
dp  = [1, 1,1,2,2,3,_  ,_] // at 7, 5/3 is smaller
dp  = [1, 1,1,2,2,3,4  ,_] // at 101, 7 is smaller
dp  = [1, 1,1,2,2,3,4  ,4] // at 18, 7 is smaller
```

# Number of LIS
