# Squarepoint Capital

## Round — Desk Quant Analyst (Date TBD, Source TBD)

**Format:** <fill in — number of questions, time limit, platform>

### Question 1 — Count Ordered Pairs With Sum in a Range
- **Difficulty:** Medium
- **Problem:** Given an array `nums`, find the total number of ordered pairs `(i, j)` such that `lower < nums[i] + nums[j] < upper`.
- **Topics:** Arrays, sorting, two-pointer
- **Notes:** Brute force is O(n²). Optimize to O(n log n):
  - A strict range condition `lower < sum < upper` splits into `(pairs with sum < upper) - (pairs with sum < lower + 1)`.
  - Sort the array, then two-pointer sweep with `l = 0, r = n-1`: if `nums[l] + nums[r] < target`, then `nums[l]` pairs validly with everything from `l+1` to `r`, i.e. `r - l` combinations — add that and move `l++`. Otherwise move `r--`.
  - This counts each unordered combination once (`l < r`), so multiply the final result by 2 to get ordered pairs `(i,j)` and `(j,i)`.
  - Use `long long` to avoid overflow.
```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    long long countOrderedPairs(vector<int>& nums, int lower, int upper) {
        sort(nums.begin(), nums.end());

        long long combinations = helper(nums, upper) - helper(nums, lower + 1);

        return combinations * 2;
    }

    long long helper(vector<int>& nums, int target) {
        int l = 0;
        int r = nums.size() - 1;
        long long count = 0;

        while (l < r) {
            if (nums[l] + nums[r] < target) {
                count += (r - l);
                l++;
            } else {
                r--;
            }
        }
        return count;
    }
};
```

### Question 2 — Palindrome Transformation
- **Difficulty:** Medium/Hard
- **Problem:** Given an array `data`, find the minimum number of operations to make it a palindrome, where symmetric pairs `(data[i], data[n-1-i])` that don't match need to be merged into the same value. Merging a connected component of size `K` takes `K - 1` operations.
- **Topics:** DSU / Union-Find, graphs
- **Notes:** Treat mismatched symmetric pairs as edges connecting values (not indices) — build a graph over values and find connected components. Since values can be up to 1e9, use `unordered_map<int,int>` for DSU parent tracking instead of an array (plain array DSU → MLE). Count an operation every time a `union` actually merges two distinct sets.
```cpp
#include <vector>
#include <unordered_map>
using namespace std;

unordered_map<int, int> parent;

int find_set(int v) {
    if (parent.find(v) == parent.end()) {
        parent[v] = v;
        return v;
    }
    if (v == parent[v]) return v;
    return parent[v] = find_set(parent[v]);
}

bool union_sets(int a, int b) {
    a = find_set(a);
    b = find_set(b);
    if (a != b) {
        parent[b] = a;
        return true;
    }
    return false;
}

int getMinOperations(vector<int> data) {
    int operations = 0;
    int n = data.size();
    for (int i = 0; i < n / 2; i++) {
        if (union_sets(data[i], data[n - 1 - i])) {
            operations++;
        }
    }
    return operations;
}
```

### Question 3 — VWAP Anomaly Alerting
- **Difficulty:** Hard
- **Problem:** Given `price[]` and `quantity[]` arrays and a list of queries `[L, R, K_min, K_max]`, compute the overall VWAP of window `[L, R]`. A sub-window of length between `K_min` and `K_max` is "defective" if its VWAP is within 5% of the overall VWAP. For each query, find the minimum number of alert points needed so every defective window contains at least one alert point.
- **Topics:** Prefix sums, greedy interval stabbing (minimum point cover for intervals)
- **Notes:** Use prefix sums for `price*quantity` and `quantity` (shift index by +1 to avoid negative-index issues) to get window VWAP in O(1). Collect all defective windows, sort by end time, and greedily place an alert at the end of the first window; skip windows already covered; whenever a window starts after the last alert, place a new alert at its end. Classic activity-selection style greedy.
```cpp
#include <vector>
#include <cmath>
#include <algorithm>
using namespace std;

struct Window { int start, end; };
bool compareWindows(Window a, Window b) { return a.end < b.end; }

vector<int> solveVWAP(vector<double>& price, vector<double>& quantity, vector<vector<int>>& queries) {
    int n = price.size();
    vector<double> num(n + 1, 0.0), den(n + 1, 0.0);

    for (int i = 0; i < n; i++) {
        num[i + 1] = num[i] + (price[i] * quantity[i]);
        den[i + 1] = den[i] + quantity[i];
    }

    vector<int> results;
    for (auto& q : queries) {
        int L = q[0], R = q[1], K_min = q[2], K_max = q[3];

        double vwap_main = (num[R + 1] - num[L]) / (den[R + 1] - den[L]);
        double max_diff = 0.05 * vwap_main;

        vector<Window> defects;
        for (int i = L; i <= R; i++) {
            for (int len = K_min; len <= K_max; len++) {
                int j = i + len - 1;
                if (j > R) break;

                double vwap_sub = (num[j + 1] - num[i]) / (den[j + 1] - den[i]);
                if (abs(vwap_sub - vwap_main) <= max_diff) {
                    defects.push_back({i, j});
                }
            }
        }

        if (defects.empty()) {
            results.push_back(0);
            continue;
        }

        sort(defects.begin(), defects.end(), compareWindows);
        int alerts = 1;
        int last_alert = defects[0].end;

        for (int i = 1; i < defects.size(); i++) {
            if (defects[i].start > last_alert) {
                alerts++;
                last_alert = defects[i].end;
            }
        }
        results.push_back(alerts);
    }
    return results;
}
```

### Question 4 — Minimum Cost to Make Pods Distinct
- **Difficulty:** Hard
- **Problem:** Given `pod[]` (ids, with possible duplicates) and `cost[]`, whenever multiple pods share an ID, all but one must move forward to the next free ID, each move incurring its `cost`. Find the minimum total cost to make all pod IDs distinct.
- **Topics:** Sweep-line, max-heap (priority_queue), DSU (alternative approach)
- **Notes:** Two viable approaches:
  - **DSU "forwarding address"**: sort descending by cost; DSU where `parent[x] = x + 1` lets you instantly jump to the next free ID, avoiding O(N²) linear scans for a free slot.
  - **Sweep-line / waiting room** (used below): sort ascending by ID; maintain a max-heap of costs for pods "waiting" at/before the current ID. At each ID, the most expensive pod claims it for free; every other pod in the heap rolls forward one step, adding its cost to the running penalty.
```cpp
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

long long minCostToMakePodsDistinct(vector<int>& pod, vector<int>& cost) {
    int n = pod.size();
    vector<pair<int, int>> pods;
    for (int i = 0; i < n; i++) pods.push_back({pod[i], cost[i]});

    sort(pods.begin(), pods.end());

    priority_queue<int> waiting_room;
    long long total_penalty = 0;
    long long waiting_sum = 0;

    int i = 0, current_id = 0;
    while (i < n || !waiting_room.empty()) {
        if (waiting_room.empty() && current_id < pods[i].first) {
            current_id = pods[i].first;
        }

        while (i < n && pods[i].first == current_id) {
            waiting_room.push(pods[i].second);
            waiting_sum += pods[i].second;
            i++;
        }

        if (!waiting_room.empty()) {
            int winner_cost = waiting_room.top();
            waiting_room.pop();
            waiting_sum -= winner_cost;
            total_penalty += waiting_sum;
        }
        current_id++;
    }
    return total_penalty;
}
```

### Question 5 — Thread Stack
- **Difficulty:** Hard
- **Problem:** Given `threadSize[]`, pick a set of non-adjacent "peak" indices (positions 1..n-2) and raise each chosen peak's value so it's strictly greater than both neighbors (cost at index `i` = `max(0, max(threadSize[i-1], threadSize[i+1]) + 1 - threadSize[i])`). Minimize total increase needed.
- **Topics:** Greedy, prefix/suffix sums, parity case split
- **Notes:**
  - If `n` is odd: only one valid peak configuration exists (indices 1, 3, 5, ...) — just sum the cost at each.
  - If `n` is even: one "gap" (skipping two elements) is required somewhere in the alternating pattern. Precompute prefix cost array (peaks from the left) and suffix cost array (peaks from the right), then try every possible gap position and take the minimum combined cost — O(N) overall.
```cpp
#include <vector>
#include <algorithm>
using namespace std;

long long findMinIncrease(vector<int>& threadSize) {
    int n = threadSize.size();
    if (n < 3) return 0;

    auto getCost = [&](int i) {
        long long target = max(threadSize[i - 1], threadSize[i + 1]) + 1;
        return max(0LL, target - threadSize[i]);
    };

    if (n % 2 != 0) {
        long long total_cost = 0;
        for (int i = 1; i < n - 1; i += 2) {
            total_cost += getCost(i);
        }
        return total_cost;
    } else {
        int k = (n - 2) / 2;
        vector<long long> pref(k + 1, 0), suff(k + 1, 0);

        for (int j = 1; j <= k; j++) {
            pref[j] = pref[j - 1] + getCost(2 * j - 1);
            suff[j] = suff[j - 1] + getCost(n - 2 * j);
        }

        long long min_total_cost = -1;
        for (int j = 0; j <= k; j++) {
            long long current_cost = pref[j] + suff[k - j];
            if (min_total_cost == -1 || current_cost < min_total_cost) {
                min_total_cost = current_cost;
            }
        }
        return min_total_cost;
    }
}
```

### Question 6 — Maximize Profit from Category Sales
- **Difficulty:** Medium/Hard
- **Problem:** Sell `n` items, each with a `category` and a `price`. The profit from selling an item equals `price * (number of distinct categories sold so far, including this item)`. Items can be sold in any order. Maximize total profit.
- **Topics:** Hash map (min per key), greedy, sorting
- **Notes:** Key insight — the multiplier only increases when you sell the *first* item from a new category, so to reach the max multiplier as cheaply as possible:
  1. **Unlock categories:** find the cheapest item in each distinct category.
  2. **Ramp up:** sort those cheapest-per-category items ascending and "sell" them first — this unlocks categories one by one while minimizing the cost paid at low multipliers.
  3. **Max profit:** once all categories are unlocked (multiplier = number of distinct categories), sell every remaining item (including the unlock items themselves, which were already counted once during ramp-up) at the max multiplier — hence the final `ans -= multiplier * minsum` correction to avoid double-counting the unlock items at the wrong rate.
```cpp
#include <vector>
#include <map>
#include <algorithm>
using namespace std;

long long maxProfit(vector<int>& category, vector<int>& price) {
    map<int, int> mpp;
    int n = category.size();

    // 1. Find the cheapest item for each category
    for (int i = 0; i < n; i++) {
        if (mpp.find(category[i]) != mpp.end()) {
            mpp[category[i]] = min(mpp[category[i]], price[i]);
        } else {
            mpp[category[i]] = price[i];
        }
    }

    // 2. Sort the minimums ascending
    vector<int> temp;
    for (auto m : mpp) {
        temp.push_back(m.second);
    }
    sort(temp.begin(), temp.end());

    long long ans = 0;
    long long minsum = 0;

    // 3. Sell the cheapest items to ramp up the multiplier
    for (int i = 0; i < temp.size(); i++) {
        ans += (i + 1) * temp[i];
        minsum += temp[i];
    }

    // 4. Sell the rest at the max multiplier
    int multiplier = temp.size();
    for (auto a : price) {
        ans += multiplier * a;
    }
    ans -= multiplier * minsum; // Remove the initial unlock items we already sold

    return ans;
}
```

### Question 7 — FIX Field Substitution
- **Difficulty:** Hard
- **Problem:** Given a target `fixTag`, a `mappings` dict (old value → new value), and a list of FIX protocol messages (`|`-delimited tag=value tokens), replace the value of `fixTag` wherever it matches a key in `mappings`. If a message changes, recompute its metadata:
  - **Tag 9 (BodyLength)**: always the 2nd field — exact character count of all fields between Tag 9 and Tag 10 (each including its trailing `|`).
  - **Tag 10 (CheckSum)**: always the last field — sum of ASCII values of every character from the start of the message through the `|` before Tag 10, mod 256, zero-padded to exactly 3 digits.
- **Topics:** String parsing/tokenizing, simulation
- **Notes:** Tokenize on `|`, skip tags 8/9 (header) and 10 (checksum, recomputed) when searching for the target tag. Only recompute BodyLength/CheckSum if a substitution actually happened (`changed` flag) — otherwise leave the message untouched. Order of operations matters: update the field → recompute BodyLength → recompute CheckSum over the *new* message → rebuild the string.
```cpp
#include <vector>
#include <string>
#include <map>
#include <sstream>
#include <iomanip>
using namespace std;

vector<string> substituteFixMessage(int fixTag, const map<string, string>& mappings, const vector<string>& FIXMessages) {
    vector<string> result;
    string targetTag = to_string(fixTag);

    for (const string& msg : FIXMessages) {
        vector<string> tokens;
        string currentToken;
        for (char c : msg) {
            if (c == '|') {
                tokens.push_back(currentToken);
                currentToken.clear();
            } else {
                currentToken += c;
            }
        }

        bool changed = false;
        for (size_t i = 2; i < tokens.size() - 1; ++i) {
            size_t equalPos = tokens[i].find('=');
            if (equalPos != string::npos) {
                string tag = tokens[i].substr(0, equalPos);
                string val = tokens[i].substr(equalPos + 1);

                if (tag == targetTag && mappings.find(val) != mappings.end()) {
                    tokens[i] = tag + "=" + mappings.at(val);
                    changed = true;
                }
            }
        }

        if (!changed) {
            result.push_back(msg);
            continue;
        }

        int newBodyLength = 0;
        for (size_t i = 2; i < tokens.size() - 1; ++i) {
            newBodyLength += tokens[i].length() + 1;
        }
        tokens[1] = "9=" + to_string(newBodyLength);

        int checksumSum = 0;
        for (size_t i = 0; i < tokens.size() - 1; ++i) {
            for (char c : tokens[i]) {
                checksumSum += c;
            }
            checksumSum += '|';
        }

        int checksum = checksumSum % 256;
        ostringstream oss;
        oss << setfill('0') << setw(3) << checksum;
        tokens.back() = "10=" + oss.str();
        // oss << setfill('0') << setw(3) << checksum  -> formats `checksum` as a string
        // padded with leading '0's to a minimum width of 3 (e.g. 7 -> "007", 42 -> "042").
        //
        // Simpler alternative if the stream syntax isn't clicking, same result:
        // string checksumStr = to_string(checksum);
        // while (checksumStr.length() < 3) {
        //     checksumStr = "0" + checksumStr;
        // }
        // tokens.back() = "10=" + checksumStr;

        string finalMsg = "";
        for (const string& token : tokens) {
            finalMsg += token + "|";
        }
        result.push_back(finalMsg);
    }
    return result;
}
```

### Overall notes
- <add: how many rounds total, what came after OA, cutoff/difficulty feel, anything else>
