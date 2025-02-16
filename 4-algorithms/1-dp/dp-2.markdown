---
title: Decision Tree DP
parent: Dynamic Programming
nav_order: 2
---
# Decision Tree DP Problems

## New 21 Game ([Link](https://leetcode.com/problems/new-21-game/))
**Topics**: probability, tree, DFS, sliding window
#### Description
We start with 0 points and draws numbers while we have less than $k$ points. During each draw, we gain an integer number of points randomly from the range $(1, max)$, where $max$ is an integer.

Each draw is independent and the outcomes have equal probabilities. We stop drawing numbers once we get $k$ or more points. We need to calculate the probability of getting less than $n$ points.

![](./new_21_game_prob.png)
#### Base Recursive Solution
Our solution is bounded by $k$, once the current score exceeds $k$, we will not take additional cards. If the current score is less or equal to $n$, we will return the current probability, else, we will return 0.
```go
if curr >= k {
	if curr <= n {
		return prob
	}
	return 0.0
}
```

If the score is not larger than $k$, we need to draw more cards and expand the tree. The decision tree has a branching factor of $max$. The probability of drawing the next card is current probability / max.

Once we have explored all the children at a node, we can sum their probabilities up. This sum is the probability of getting a score less than $n$ **if we chose this node, at the current level**, as shown by the code and diagram below.

```go
childProbSum := 0
for i := 1; i < max; i ++ {
	childProbSum += dfs(curr+i, prob/float64(max))
}
```

![](./new_21_game_base_case.png)

#### Optimal Substructure

Each node in the tree represents the probability of getting a *good score* that's larger or equal $k$ and smaller or equal $n$. The global solution can be found by summing the result of all sub-trees.

#### Overlapping Subproblem

As we explore different nodes in the recursion tree, we will definitely encounter nodes sharing the same value. For example, the two highlighted subtrees have identical structure.

![](new_21_game_overlap.png)

Instead of recomputing the same problem, we can store the result for **score=2** in a cache. However, based on the depth of the node, the probability will differ. Hence, we should revert back to the **base** probability when storing the result in the table with `baseChildProbSum / prob`, which treats the node as if its the root.

```go
baseChildProbSum := 0.0
for i := 1; i <= maxPts; i++ {
	childProb := dfs(curr+i, prob/float64(maxPts))
	baseChildProbSum += childProb
}
memo[curr] = baseChildProbSum / prob
```

The complete DFS solution will be
```go
func new21Game(n int, k int, maxPts int) float64 {
	if k == 0 {
		return 1.0
	}
	memo := map[int]float64{}
	var dfs func(curr int, prob float64) float64
	dfs = func(curr int, prob float64) float64 {
		if curr >= k {
			if curr <= n {
				return prob
			}
			return 0.0
		}
		if prob, exists := memo[0]; exists {
			return prob
		}
		baseChildProbSum := 0.0
		for i := 1; i <= maxPts; i++ {
			childProb := dfs(curr+i, prob/float64(maxPts))
			baseChildProbSum += childProb
		}
		memo[curr] = baseChildProbSum / prob
		return baseChildProbSum
	}
	dfs(0, 1)
	return memo[0]
}
```
