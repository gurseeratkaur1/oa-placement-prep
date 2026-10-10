# Spotnana

## Round — OA (2026-10-10, Source TBD)

**Format:** <fill in — number of questions, time limit, platform>

### Question 1 — Minimize Tree Depth
- **Difficulty:** Hard
- **Problem:** Given a tree with `tree_nodes` nodes (indexed 1 to `tree_nodes`), rooted at Node 1, edges given as `tree_from` and `tree_to` arrays, and an integer `maxOperations`. In one operation, detach any node from its current parent and reattach it directly to the root (Node 1) — its entire subtree moves with it. Find the minimum possible depth (max height) of the tree achievable using at most `maxOperations` operations.
- **Topics:** Trees, binary search on the answer, greedy
- **Notes:** Binary search on the answer `D` (candidate max depth), range `[1, originalHeight]`. Feasibility check for a fixed `D`: BFS from the root (root depth = 1); whenever a node's natural depth (`parent_depth + 1`) would exceed `D`, cut it — reattach to root, reset its depth to 2, count one operation — and continue the BFS using each node's *post-cut* depth for its own children. Feasible iff total cuts ≤ `maxOperations`. Optimal because cutting a node the moment it first exceeds `D` fixes its whole subtree in one operation; delaying would force cutting multiple descendants individually instead. Use BFS, not recursive DFS — avoids stack overflow on a skewed/deep tree.
```cpp
#include <vector>
using namespace std;

bool feasible(int D, long long maxOperations, vector<vector<int>>& adj, int n) {
    vector<int> depth(n + 1, 0);
    vector<bool> visited(n + 1, false);
    vector<int> q; q.reserve(n);
    q.push_back(1);
    visited[1] = true;
    depth[1] = 1;

    long long ops = 0;
    for (int head = 0; head < (int)q.size(); head++) {
        int u = q[head];
        for (int v : adj[u]) {
            if (visited[v]) continue;
            visited[v] = true;
            int d = depth[u] + 1;
            if (d > D) {
                ops++;
                d = 2; // reattached directly under root
            }
            depth[v] = d;
            q.push_back(v);
        }
    }
    return ops <= maxOperations;
}

int minimizeDepth(int tree_nodes, vector<int>& tree_from, vector<int>& tree_to, long long maxOperations) {
    vector<vector<int>> adj(tree_nodes + 1);
    for (size_t i = 0; i < tree_from.size(); i++) {
        adj[tree_from[i]].push_back(tree_to[i]);
        adj[tree_to[i]].push_back(tree_from[i]);
    }

    int low = 1, high = tree_nodes, ans = tree_nodes;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (feasible(mid, maxOperations, adj, tree_nodes)) {
            ans = mid;
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    return ans;
}
```

### Question 2 — Maximize Profit with Bitwise OR Constraint
- **Difficulty:** Hard
- **Problem:** Given arrays `id` and `profit` (same length) representing tasks — task `i` has ID `id[i]` and profit `profit[i]` — and an integer `k`. Choose a subset of tasks such that the bitwise OR of all chosen task IDs is `<= k`. Return the maximum total profit achievable.
- **Topics:** Bitmask DP, digit DP on bits
- **Notes:** The naive idea ("only keep tasks whose `id` is a bitwise subset of `k`") is too strict — e.g. `k=6 (110)`, `id=5 (101)` isn't a subset of `k`'s bits, yet `5 ≤ 6` is trivially fine standing alone. Correct characterization: any `X ≤ k` either equals `k` exactly, or there's some bit `p` where `k` has a 1, `X` has a 0, and everything above bit `p` is a subset of `k`'s bits (bits below `p` are then free, since a strictly smaller bit at a higher position already guarantees `X < k` regardless of what's below). So for each candidate break bit `p` (every bit where `k` is 1) plus the "fully subset of `k`" case, filter tasks whose `id` matches that bit pattern and sum **all** qualifying profits (no further constraint needed once the filter holds — adding every qualifying task is always safe). Take the max over all candidates. O(n · bits).
```cpp
#include <vector>
using namespace std;

long long maxProfitWithORConstraint(vector<long long>& id, vector<long long>& profit, long long k) {
    int n = id.size();
    long long best = 0;

    // Candidate: id itself is a bitwise subset of k (covers OR == k too)
    {
        long long sum = 0;
        for (int i = 0; i < n; i++)
            if ((id[i] & ~k) == 0) sum += profit[i];
        best = max(best, sum);
    }

    // Candidate: break at bit p where k has a 1 — bits above p subset of k, bit p forced 0
    for (int p = 0; p < 62; p++) {
        if (!((k >> p) & 1)) continue;
        long long highMask = ~((1LL << (p + 1)) - 1);
        long long sum = 0;
        for (int i = 0; i < n; i++) {
            bool bitPZero = !((id[i] >> p) & 1);
            bool highSubset = ((id[i] & highMask) & ~k) == 0;
            if (bitPZero && highSubset) sum += profit[i];
        }
        best = max(best, sum);
    }

    return best;
}
```

### Question 3 — Minimize Deletion Penalty
- **Difficulty:** Medium/Hard
- **Problem:** Given strings `A` and `B`, make `B` a subsequence of `A` by deleting characters from `B` (deleted positions need not be contiguous). Let `min_index`/`max_index` be the smallest/largest original index deleted from `B`. Penalty = `(max_index - min_index) + 1`. Find the minimum possible penalty (0 if `B` is already a subsequence of `A`).
- **Topics:** Two pointers, binary search, subsequence matching
- **Notes:** Penalty only depends on the *span* between the first and last deleted index, not how many you delete — so for any candidate span, the optimal move is to delete the entire contiguous window `[l, r]` from `B` (keeping anything inside only adds a matching constraint without lowering the cost). This reduces to: find the shortest contiguous substring to delete from `B` so the remainder is a subsequence of `A`.
  - Precompute `prefixPos[i]` = fewest characters of `A` (from the left) needed to match `B[0..i)` as a subsequence (greedy two-pointer).
  - Precompute `suffixPos[j]` = starting index in `A` such that `B[j..m)` matches `A[suffixPos[j]..n)` (greedy two-pointer from the right).
  - Both arrays are monotonic in their index, so for each prefix length `l`, find the smallest `j >= l` with `suffixPos[j] >= prefixPos[l]` (meaning the two matched regions of `A` don't overlap) via a single two-pointer sweep over `l` — no re-scanning needed. Window length `j - l` is the penalty for that split; take the min over all `l`. O(|A| + |B|) overall. Deleting all of `B` (window = whole string) is always a valid fallback, since the empty string is trivially a subsequence of anything.
```cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int minDeletionPenalty(const string& A, const string& B) {
    int n = A.size(), m = B.size();
    const int INF = n + 1;

    vector<int> prefixPos(m + 1, INF);
    prefixPos[0] = 0;
    {
        int ai = 0;
        for (int bi = 0; bi < m; bi++) {
            while (ai < n && A[ai] != B[bi]) ai++;
            if (ai == n) break;
            ai++;
            prefixPos[bi + 1] = ai;
        }
    }

    vector<int> suffixPos(m + 1, INF);
    suffixPos[m] = n;
    {
        int ai = n - 1;
        for (int bi = m - 1; bi >= 0; bi--) {
            while (ai >= 0 && A[ai] != B[bi]) ai--;
            if (ai < 0) break;
            suffixPos[bi] = ai;
            ai--;
        }
    }

    int best = m; // deleting all of B is always valid
    int j = 0;
    for (int l = 0; l <= m; l++) {
        if (prefixPos[l] == INF) break;
        if (j < l) j = l;
        while (j <= m && (suffixPos[j] == INF || suffixPos[j] < prefixPos[l])) j++;
        if (j <= m) best = min(best, j - l);
    }

    return best; // 0 means B is already a subsequence of A
}
```

### Overall notes
- <add: how many rounds total, what came after OA, cutoff/difficulty feel, anything else>
