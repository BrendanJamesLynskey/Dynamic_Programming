# Dynamic Programming

**Computer Science Fundamentals Series**

Optimal substructure · Memoization · Tabulation · Knapsack · Edit distance · State transitions

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [What Is Dynamic Programming?](#slide-02--what-is-dynamic-programming)
2. [Memoization vs Tabulation](#slide-03--memoization-vs-tabulation)
3. [Fibonacci: The Introductory Example](#slide-04--fibonacci-the-introductory-example)
4. [The DP Recipe](#slide-05--the-dp-recipe)
5. [1D DP: Coin Change](#slide-06--1d-dp-coin-change)
6. [1D DP: Longest Increasing Subsequence](#slide-07--1d-dp-longest-increasing-subsequence)
7. [2D DP: Edit Distance](#slide-08--2d-dp-edit-distance-levenshtein)
8. [2D DP: Longest Common Subsequence](#slide-09--2d-dp-longest-common-subsequence)
9. [2D DP: Matrix Chain Multiplication](#slide-10--2d-dp-matrix-chain-multiplication)
10. [Knapsack Problems](#slide-11--knapsack-problems)
11. [DP on Strings](#slide-12--dp-on-strings)
12. [DP on Trees](#slide-13--dp-on-trees)
13. [Bitmask DP](#slide-14--bitmask-dp)
14. [Interval DP](#slide-15--interval-dp)
15. [Space Optimisation Techniques](#slide-16--space-optimisation-techniques)
16. [DP Optimisation Techniques](#slide-17--dp-optimisation-techniques)
17. [Identifying DP Problems](#slide-18--identifying-dp-problems)
18. [Common Pitfalls & Debugging](#slide-19--common-pitfalls--debugging)
19. [Summary & Further Reading](#slide-20--summary--further-reading)

---

## Slide 02 -- What Is Dynamic Programming?

Dynamic programming (DP) is an algorithmic paradigm that solves complex problems by breaking them into simpler overlapping subproblems, solving each once, and storing the result for reuse.

### Optimal Substructure

An optimal solution to the problem contains optimal solutions to its subproblems. If you can express the answer in terms of answers to smaller instances of the same problem, the structure is present.

Example: shortest path from A to C via B = shortest(A→B) + shortest(B→C)

### Overlapping Subproblems

The same subproblem is encountered many times during recursion. Naive recursion recomputes them exponentially; DP computes each exactly once.

Example: fib(5) calls fib(3) twice, fib(2) three times

> **When both properties hold** -- optimal substructure and overlapping subproblems -- dynamic programming applies. Without overlap, divide-and-conquer suffices. Without optimal substructure, greedy or DP will not yield correct results.

---

## Slide 03 -- Memoization vs Tabulation

### Top-Down (Memoization)

- Start from the original problem, recurse into subproblems
- Cache results in a hash map or array on first computation
- Return cached value on subsequent calls
- Only solves subproblems that are actually needed
- Uses the call stack -- risk of stack overflow for deep recursion

```python
memo = {}
def fib(n):
    if n <= 1: return n
    if n in memo: return memo[n]
    memo[n] = fib(n-1) + fib(n-2)
    return memo[n]
```

### Bottom-Up (Tabulation)

- Start from the smallest subproblems, build up iteratively
- Fill a table (array) in a defined order
- No recursion -- no stack overflow risk
- Solves all subproblems, even those not strictly needed
- Often enables space optimisation (rolling arrays)

```python
def fib(n):
    if n <= 1: return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

> **Rule of thumb:** use top-down when the state space is sparse or hard to enumerate; use bottom-up when you need every subproblem and want to optimise space.

---

## Slide 04 -- Fibonacci: The Introductory Example

### Naive recursion -- O(2^n)

The recursion tree explodes exponentially. `fib(40)` makes over one billion calls.

```python
def fib(n):
    if n <= 1: return n
    return fib(n-1) + fib(n-2)
```

### Complexity comparison

| Approach | Time | Space |
|----------|------|-------|
| Naive recursion | `O(2^n)` | `O(n)` stack |
| Memoization | `O(n)` | `O(n)` |
| Tabulation | `O(n)` | `O(n)` |
| Space-optimised | `O(n)` | `O(1)` |

### Space-optimised solution

```python
def fib(n):
    if n <= 1: return n
    prev2, prev1 = 0, 1
    for _ in range(2, n + 1):
        prev2, prev1 = prev1, prev2 + prev1
    return prev1
```

> **Key insight:** we only ever need the last two values -- no need to store the entire table. This pattern generalises to many 1D DP problems.

---

## Slide 05 -- The DP Recipe

A systematic approach to solving any DP problem:

1. **Define the state** -- what variables uniquely describe a subproblem? `dp[i]`, `dp[i][j]`, `dp[mask]`
2. **Write the recurrence** -- express `dp[state]` in terms of smaller states. This is the transition function.
3. **Identify base cases** -- the smallest subproblems with known answers. `dp[0] = 0`, `dp[0][0] = 1`, etc.
4. **Determine iteration order** -- ensure all dependencies are computed before the current state.
5. **Extract the answer** -- locate the final state: `dp[n]`, `max(dp)`, or backtrack for the solution path.

### Example: Climbing Stairs

You can climb 1 or 2 steps at a time. How many distinct ways to reach step `n`?

```python
# State: dp[i] = ways to reach step i
# Recurrence: dp[i] = dp[i-1] + dp[i-2]
# Base: dp[0] = 1, dp[1] = 1
# Order: left to right
# Answer: dp[n]

def climb_stairs(n):
    if n <= 1: return 1
    prev2, prev1 = 1, 1
    for i in range(2, n + 1):
        prev2, prev1 = prev1, prev2 + prev1
    return prev1
```

> **Notice:** this is exactly the Fibonacci sequence shifted by one index. Many 1D DP problems reduce to similar linear recurrences.

---

## Slide 06 -- 1D DP: Coin Change

Given coins of denominations `coins[]` and a target amount, find the minimum number of coins needed.

### Recurrence

```python
dp[0] = 0  # base case
dp[amount] = min(dp[amount - coin] + 1)
             for coin in coins
             if amount - coin >= 0
```

### Full solution

```python
def coin_change(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a:
                dp[a] = min(dp[a], dp[a - c] + 1)
    return dp[amount] if dp[amount] != float('inf') else -1
```

### Trace: coins = [1, 3, 4], amount = 6

| amount | dp[amount] | best coin |
|--------|-----------|-----------|
| 0 | 0 | -- |
| 1 | 1 | 1 |
| 2 | 2 | 1 |
| 3 | 1 | 3 |
| 4 | 1 | 4 |
| 5 | 2 | 1+4 |
| 6 | 2 | 3+3 |

> **Complexity:** Time `O(amount * |coins|)`, Space `O(amount)`. Greedy fails here -- greedy picks 4+1+1 = 3 coins, but DP finds 3+3 = 2 coins.

---

## Slide 07 -- 1D DP: Longest Increasing Subsequence

Given an array `nums[]`, find the length of the longest strictly increasing subsequence (LIS).

### O(n^2) DP approach

```python
# State: dp[i] = length of LIS ending at index i
# Recurrence: dp[i] = max(dp[j] + 1)
#             for all j < i where nums[j] < nums[i]
# Base: dp[i] = 1 for all i

def lis(nums):
    n = len(nums)
    dp = [1] * n
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
```

### O(n log n) with patience sorting

```python
import bisect

def lis_fast(nums):
    tails = []  # tails[i] = smallest tail of
                # all increasing subseqs of length i+1
    for x in nums:
        pos = bisect.bisect_left(tails, x)
        if pos == len(tails):
            tails.append(x)
        else:
            tails[pos] = x
    return len(tails)
```

### Trace: [10, 9, 2, 5, 3, 7, 101, 18]

| Step | tails | LIS length |
|------|-------|-----------|
| 10 | [10] | 1 |
| 9 | [9] | 1 |
| 2 | [2] | 1 |
| 5 | [2, 5] | 2 |
| 3 | [2, 3] | 2 |
| 7 | [2, 3, 7] | 3 |
| 101 | [2, 3, 7, 101] | 4 |
| 18 | [2, 3, 7, 18] | 4 |

---

## Slide 08 -- 2D DP: Edit Distance (Levenshtein)

Minimum insertions, deletions, and substitutions to transform string `a` into string `b`.

### Recurrence

```python
# State: dp[i][j] = edit distance between a[0..i-1] and b[0..j-1]

if a[i-1] == b[j-1]:
    dp[i][j] = dp[i-1][j-1]       # match
else:
    dp[i][j] = 1 + min(
        dp[i-1][j],     # delete from a
        dp[i][j-1],     # insert into a
        dp[i-1][j-1]    # substitute
    )
```

### Full solution

```python
def edit_distance(a, b):
    m, n = len(a), len(b)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(m+1): dp[i][0] = i
    for j in range(n+1): dp[0][j] = j
    for i in range(1, m+1):
        for j in range(1, n+1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j],
                    dp[i][j-1], dp[i-1][j-1])
    return dp[m][n]
```

> **Answer for "kitten" → "sitting": 3** -- substitute k→s, substitute e→i, insert g. Time `O(mn)`, Space `O(mn)` reducible to `O(min(m,n))`.

---

## Slide 09 -- 2D DP: Longest Common Subsequence

Find the length of the longest subsequence common to two strings `a` and `b`. Characters need not be contiguous but must preserve order.

### Recurrence

```python
# State: dp[i][j] = LCS length of a[0..i-1], b[0..j-1]

if a[i-1] == b[j-1]:
    dp[i][j] = dp[i-1][j-1] + 1
else:
    dp[i][j] = max(dp[i-1][j], dp[i][j-1])
```

### Solution

```python
def lcs(a, b):
    m, n = len(a), len(b)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(1, m+1):
        for j in range(1, n+1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```

### Recovering the LCS string

```python
def recover_lcs(a, b, dp):
    i, j = len(a), len(b)
    result = []
    while i > 0 and j > 0:
        if a[i-1] == b[j-1]:
            result.append(a[i-1])
            i -= 1; j -= 1
        elif dp[i-1][j] > dp[i][j-1]:
            i -= 1
        else:
            j -= 1
    return ''.join(reversed(result))
```

> **Applications:** diff utilities, version control, DNA sequence alignment, plagiarism detection.

---

## Slide 10 -- 2D DP: Matrix Chain Multiplication

Given matrices A1...An with dimensions p[0]×p[1], p[1]×p[2], ..., p[n-1]×p[n], find the parenthesisation that minimises total scalar multiplications.

### Recurrence

```python
# State: dp[i][j] = min cost to multiply A_i..A_j
dp[i][j] = min over k in [i, j):
    dp[i][k] + dp[k+1][j] + p[i-1]*p[k]*p[j]

# Base: dp[i][i] = 0 (single matrix, no cost)
```

### Solution

```python
def matrix_chain(p):
    n = len(p) - 1
    dp = [[0]*n for _ in range(n)]
    for length in range(2, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            dp[i][j] = float('inf')
            for k in range(i, j):
                cost = (dp[i][k] + dp[k+1][j]
                        + p[i]*p[k+1]*p[j+1])
                dp[i][j] = min(dp[i][j], cost)
    return dp[0][n-1]
```

### Example: p = [10, 30, 5, 60]

| Parenthesisation | Cost |
|------------------|------|
| (A1·A2)·A3 | 10·30·5 + 10·5·60 = **4500** |
| A1·(A2·A3) | 30·5·60 + 10·30·60 = **27000** |

> **6× difference!** The wrong parenthesisation can be catastrophically expensive. This is a classic interval DP -- iterate by increasing interval length.

---

## Slide 11 -- Knapsack Problems

### 0/1 Knapsack

Each item can be taken at most once.

```python
# dp[i][w] = max value using items 0..i with capacity w
if w[i] <= j:
  dp[i][j] = max(
    dp[i-1][j],             # skip
    dp[i-1][j-w[i]] + v[i]  # take
  )
# Time: O(nW), Space: O(nW) → O(W)
```

NP-hard -- pseudo-polynomial in W.

### Unbounded Knapsack

Each item has unlimited supply.

```python
# dp[w] = max value with capacity w
for w in range(1, W + 1):
  for i in range(n):
    if wt[i] <= w:
      dp[w] = max(dp[w], dp[w - wt[i]] + val[i])
# Time: O(nW), Space: O(W)
```

Coin change is a variant of this.

### Fractional Knapsack

Items can be divided -- take any fraction. Greedy by value/weight ratio works. Time: O(n log n). No DP needed.

> **Key distinction:** fractional knapsack has the greedy-choice property. 0/1 and unbounded do not -- you must consider all combinations via DP. The 0/1 variant's 1D space optimisation requires iterating capacity *right to left* to avoid using an item twice.

---

## Slide 12 -- DP on Strings

Strings are a rich domain for DP. The state is typically indices into one or two strings.

### Longest Palindromic Subsequence

```python
# dp[i][j] = LPS length of s[i..j]
if s[i] == s[j]:
    dp[i][j] = dp[i+1][j-1] + 2
else:
    dp[i][j] = max(dp[i+1][j], dp[i][j-1])
# Time: O(n^2), Space: O(n^2) → O(n)
```

### Palindrome Partitioning

```python
# dp[i] = min cuts for s[0..i] to be all palindromes
dp[i] = min(dp[j-1] + 1)
        for j in [0..i] where s[j..i] is a palindrome
# Pre-compute is_pal[i][j] with DP
# Time: O(n^2), Space: O(n^2)
```

### Distinct Subsequences

Count the number of distinct subsequences of `s` that equal `t`.

```python
# dp[i][j] = ways to form t[0..j-1] from s[0..i-1]
if s[i-1] == t[j-1]:
    dp[i][j] = dp[i-1][j-1] + dp[i-1][j]
else:
    dp[i][j] = dp[i-1][j]
# Time: O(mn), Space: O(mn) → O(n)
```

> **Pattern:** string DP almost always uses `dp[i][j]` indexed by positions in one or two strings. The transition considers match/mismatch at the current character.

---

## Slide 13 -- DP on Trees

Tree DP computes values at each node based on its children, via post-order DFS. The state is typically `dp[node]` or `dp[node][choice]`.

### Maximum Independent Set

Select nodes with maximum total weight such that no two adjacent nodes are selected.

```python
# dp[v][0] = max if v is NOT selected
# dp[v][1] = max if v IS selected

dp[v][0] = sum(max(dp[c][0], dp[c][1])
               for c in children(v))

dp[v][1] = weight[v] +
           sum(dp[c][0] for c in children(v))

# Answer: max(dp[root][0], dp[root][1])
```

### Tree Diameter

```python
# dp[v] = longest path starting at v going downward
dp[v] = max(dp[c] + 1 for c in children(v))
# At each node, diameter candidate = sum of two longest downward paths
diameter = max(top1 + top2) across all nodes
```

### Rerooting Technique

Some tree DP problems ask for the answer rooted at *every* node. Naive approach: run DFS from each node -- O(n^2). Rerooting: compute DP rooted at node 1, then "shift the root" to each adjacent node in O(1) using precomputed values. Total: O(n).

> **Key insight:** tree DP works because trees have no cycles -- each subtree is an independent subproblem. Process leaves first, work upward.

---

## Slide 14 -- Bitmask DP

When the state involves a subset of n elements (n ≤ 20--25), represent the subset as a bitmask integer. State: `dp[mask]` or `dp[mask][i]`.

### Travelling Salesman Problem (TSP)

```python
# dp[mask][i] = min cost to visit cities in 'mask' ending at city i
# Base: dp[1 << start][start] = 0

for mask in range(1, 1 << n):
    for i in range(n):
        if not (mask & (1 << i)): continue
        for j in range(n):
            if mask & (1 << j): continue
            new_mask = mask | (1 << j)
            dp[new_mask][j] = min(
                dp[new_mask][j],
                dp[mask][i] + dist[i][j]
            )

# Answer: min(dp[(1<<n)-1][i] + dist[i][start])
# Time: O(2^n * n^2), Space: O(2^n * n)
```

### Bit manipulation essentials

| Operation | Code |
|-----------|------|
| Set bit i | `mask \| (1 << i)` |
| Clear bit i | `mask & ~(1 << i)` |
| Check bit i | `mask & (1 << i)` |
| Toggle bit i | `mask ^ (1 << i)` |
| Count set bits | `bin(mask).count('1')` |
| Lowest set bit | `mask & (-mask)` |
| Iterate subsets of mask | `sub = mask; while sub: ... sub = (sub-1) & mask` |

---

## Slide 15 -- Interval DP

Interval DP solves problems where the state is a contiguous range `[i, j]` and the answer for a range depends on splitting it into smaller ranges.

### Pattern

```python
for length in range(2, n + 1):
    for i in range(n - length + 1):
        j = i + length - 1
        for k in range(i, j):  # split point
            dp[i][j] = combine(dp[i][k], dp[k+1][j], cost(i,k,j))
```

### Burst Balloons

```python
# dp[i][j] = max coins from bursting balloons i..j
dp[i][j] = max over k in [i..j]:
    dp[i][k-1] + dp[k+1][j] + nums[i-1] * nums[k] * nums[j+1]
# Key trick: k is the LAST balloon burst in the range, not the first
```

### Optimal BST

```python
# dp[i][j] = min cost BST for keys i..j
dp[i][j] = min over r in [i..j]:
    dp[i][r-1] + dp[r+1][j] + sum(freq[i..j])
# Time: O(n^3), improved to O(n^2) with Knuth's optimisation
```

### Classic interval DP problems

- Matrix chain multiplication
- Minimum cost polygon triangulation
- Palindrome partitioning
- Burst balloons
- Optimal BST / Knuth's optimisation
- Stone game / merging stones

---

## Slide 16 -- Space Optimisation Techniques

### Rolling array (two rows)

When `dp[i]` depends only on `dp[i-1]`, keep just two rows and alternate.

```python
# Edit distance: O(mn) → O(n) space
def edit_distance(a, b):
    m, n = len(a), len(b)
    prev = list(range(n + 1))
    curr = [0] * (n + 1)
    for i in range(1, m + 1):
        curr[0] = i
        for j in range(1, n + 1):
            if a[i-1] == b[j-1]:
                curr[j] = prev[j-1]
            else:
                curr[j] = 1 + min(prev[j], curr[j-1], prev[j-1])
        prev, curr = curr, prev
    return prev[n]
```

### In-place update (1D knapsack)

For 0/1 knapsack, iterate capacity **right to left** to avoid counting an item twice.

```python
dp = [0] * (W + 1)
for i in range(n):
    for w in range(W, wt[i] - 1, -1):
        dp[w] = max(dp[w], dp[w - wt[i]] + val[i])
```

For unbounded knapsack, iterate **left to right** -- reusing the same item is allowed.

### Summary of reductions

| Problem | Full | Optimised |
|---------|------|-----------|
| Fibonacci | O(n) | O(1) |
| LCS / Edit dist | O(mn) | O(min(m,n)) |
| 0/1 Knapsack | O(nW) | O(W) |
| Grid paths | O(mn) | O(n) |
| LIS | O(n^2) | O(n) |

---

## Slide 17 -- DP Optimisation Techniques

### Knuth's Optimisation

For interval DP where the optimal split point `opt[i][j]` satisfies `opt[i][j-1] ≤ opt[i][j] ≤ opt[i+1][j]`, reduce from O(n^3) to O(n^2). Applies to: optimal BST, certain stone-merging variants.

### Divide & Conquer Optimisation

When `dp[i][j] = min over k (dp[i-1][k] + C(k,j))` and the optimal k increases with j, use D&C to reduce one layer from O(n^2) to O(n log n). Applies to: many partition-into-k-groups problems.

### Convex Hull Trick

When the recurrence has the form `dp[i] = min(dp[j] + b[j]*a[i])` and slopes `b[j]` are monotone, maintain a convex hull of lines. Query is O(1) amortised. Li Chao tree generalises to arbitrary slope order.

### Monotone Queue / Deque

For sliding-window minimum/maximum in DP transitions. Reduces per-transition cost from O(k) to O(1) amortised.

### SOS DP (Sum over Subsets)

Compute for every bitmask the sum over all its submasks. Naive is O(3^n); SOS DP does it in O(n · 2^n).

```python
for bit in range(n):
    for mask in range(1 << n):
        if mask & (1 << bit):
            dp[mask] += dp[mask ^ (1 << bit)]
```

> **When to use:** these optimisations matter in competitive programming and at scale. In interviews, the standard O(n^2) or O(n^3) DP is almost always sufficient.

---

## Slide 18 -- Identifying DP Problems

### Signal phrases in problem statements

| Phrase | Likely DP type |
|--------|---------------|
| "minimum / maximum" | Optimisation DP |
| "count the number of ways" | Counting DP |
| "is it possible to..." | Boolean DP (feasibility) |
| "longest / shortest" | Sequence DP |
| "partition into groups" | Interval / subset DP |
| "subsequence" | 1D or 2D string DP |
| "grid / matrix traversal" | 2D grid DP |
| "subset / bitmask" | Bitmask DP (n ≤ 20) |

### DP vs other paradigms

| Paradigm | When to use |
|----------|-------------|
| **Greedy** | Greedy-choice property holds; locally optimal = globally optimal |
| **D&C** | Subproblems are independent (no overlap) |
| **DP** | Overlapping subproblems + optimal substructure |
| **Backtracking** | Need all solutions, not just optimal; pruning helps |

### Decision checklist

- Can I express the answer as a function of smaller inputs?
- Do subproblems repeat? (Draw the recursion tree)
- Does the state space fit in memory?
- Can I define a clear recurrence relation?
- Is the iteration order well-defined?

---

## Slide 19 -- Common Pitfalls & Debugging

### Pitfalls

- **Wrong base case** -- Off-by-one errors in base cases propagate through the entire table. Always verify: what happens when the input is empty? When it has one element?
- **Wrong iteration order** -- If `dp[i]` depends on `dp[i-1]` and `dp[i+1]`, a single left-to-right pass won't work. Draw the dependency graph.
- **State not fully captured** -- Missing a dimension in the state leads to incorrect transitions. Example: forgetting to track "last element chosen" in problems requiring it.
- **Integer overflow** -- Counting problems can produce astronomically large values. Use modular arithmetic (`% 10^9+7`) or big integers as needed.

### Debugging strategies

- **Print the DP table** -- For small inputs, print the entire table and manually verify each cell against the recurrence.
- **Start with brute force** -- Write the naive recursive solution first. Verify it works. Then add memoization. This separates correctness from optimisation.
- **Test edge cases** -- Empty input, single element, all elements identical, already sorted / reverse sorted, maximum constraints.
- **Stress test** -- Generate random small inputs, compare brute-force vs DP output. If they diverge, you have a minimal failing case to debug.

---

## Slide 20 -- Summary & Further Reading

### Key takeaways

- DP applies when a problem has **optimal substructure** and **overlapping subproblems**
- Top-down (memoization) and bottom-up (tabulation) are equivalent in power -- choose based on the problem's structure
- The five-step recipe -- state, recurrence, base case, iteration order, answer extraction -- works universally
- 1D, 2D, interval, tree, bitmask, and string DP cover the vast majority of problems
- Space optimisation (rolling arrays, in-place updates) often reduces memory by an order of magnitude
- Always start with brute-force recursion, verify correctness, then optimise with memoization or tabulation
- Practice pattern recognition -- most DP problems map to a small set of well-known templates

### Recommended reading

| Source | Description |
|--------|------------|
| **CLRS** | *Introduction to Algorithms* -- chapters 14--15 cover DP foundations rigorously |
| **Kleinberg & Tardos** | *Algorithm Design* -- excellent intuition-building DP chapter |
| **Competitive Programmer's Handbook** | Laaksonen -- free PDF, concise DP coverage with contest examples |
| **LeetCode** | [leetcode.com/tag/dynamic-programming](https://leetcode.com/tag/dynamic-programming/) -- 500+ problems sorted by difficulty |
| **Codeforces EDU** | [DP section](https://codeforces.com/edu/courses) -- structured problems with editorials |
| **MIT 6.006** | Erik Demaine's lectures on DP -- free on YouTube, exceptional clarity |

> **Practice path:** Fibonacci → climbing stairs → coin change → LCS → edit distance → knapsack → LIS → interval DP → tree DP → bitmask DP. Master each level before advancing.
