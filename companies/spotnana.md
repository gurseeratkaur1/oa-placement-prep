# Spotnana

## Round — OA (2026-10-10, Source TBD)

**Format:** <fill in — number of questions, time limit, platform>

### Question 1 — Minimize Tree Depth
- **Difficulty:** Hard
- **Problem:** Given a tree with `tree_nodes` nodes (indexed 1 to `tree_nodes`), rooted at Node 1, edges given as `tree_from` and `tree_to` arrays, and an integer `maxOperations`. In one operation, detach any node from its current parent and reattach it directly to the root (Node 1) — its entire subtree moves with it. Find the minimum possible depth (max height) of the tree achievable using at most `maxOperations` operations.
- **Topics:** Trees, binary search on the answer, greedy
- **Notes:** <to discuss>

### Question 2 — Maximize Profit with Bitwise OR Constraint
- **Difficulty:** Hard
- **Problem:** Given arrays `id` and `profit` (same length) representing tasks — task `i` has ID `id[i]` and profit `profit[i]` — and an integer `k`. Choose a subset of tasks such that the bitwise OR of all chosen task IDs is `<= k`. Return the maximum total profit achievable.
- **Topics:** Bitmask DP, digit DP on bits
- **Notes:** <to discuss>

### Question 3 — Minimize Deletion Penalty
- **Difficulty:** Medium/Hard
- **Problem:** Given strings `A` and `B`, make `B` a subsequence of `A` by deleting characters from `B` (deleted positions need not be contiguous). Let `min_index`/`max_index` be the smallest/largest original index deleted from `B`. Penalty = `(max_index - min_index) + 1`. Find the minimum possible penalty (0 if `B` is already a subsequence of `A`).
- **Topics:** Two pointers, binary search, subsequence matching
- **Notes:** <to discuss>

### Overall notes
- <add: how many rounds total, what came after OA, cutoff/difficulty feel, anything else>
