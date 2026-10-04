# Squarepoint Capital

## Round — Desk Quant Analyst (Date TBD, Source TBD)

**Format:** <fill in — number of questions, time limit, platform>

### Question 1 — Count pairs with sum in range
- **Difficulty:** Medium
- **Problem:** Given an array `arr`, find the total number of ordered pairs `(i, j)` such that `lower_limit < arr[i] + arr[j] < upper_limit`.
- **Topics:** Arrays, sorting, two-pointer, binary search
- **Notes:** Brute force is O(n²) (check every pair). Optimize to O(n log n):
  - Sort the array first.
  - For each `i`, binary search for the range of `j` satisfying the sum bounds, or
  - Use two pointers moving inward after sorting (one pointer tracks upper bound, one tracks lower bound) to count valid pairs in a single pass per boundary.
  - Watch for: ordered vs. unordered pairs (does `(i,j)` and `(j,i)` both count, and is `i == j` allowed?), and whether bounds are strict (`<`) or inclusive (`<=`).

### Overall notes
- <add: how many rounds total, what came after OA, cutoff/difficulty feel, anything else>
