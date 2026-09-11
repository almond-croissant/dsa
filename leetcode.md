# Increasing Triplet Subsequence

## Logic

We need to find **3 elements in increasing order**:

```text
first < second < third
```

while maintaining their original order.

Instead of checking every possible triplet, maintain two variables:

* `first` → smallest value seen so far
* `second` → smallest possible value that is greater than `first`

For every `num`:

1. If `num <= first`, update `first`.
2. Else if `num <= second`, update `second`.
3. Else, `num > second`, meaning we have:

```text
first < second < num
```

So an increasing triplet exists → return `true`.

If we finish the array without finding one → return `false`.

### Example

```text
nums = [2, 1, 5, 0, 4, 6]

first = 2
first = 1
second = 5
first = 0
second = 4
6 > 4  → triplet found
```

Therefore:

```text
0 < 4 < 6
```

## C++ Code

```cpp
class Solution {
public:
    bool increasingTriplet(vector<int>& nums) {
        int first = INT_MAX;
        int second = INT_MAX;

        for (int num : nums) {

            if (num <= first) {
                first = num;
            }
            else if (num <= second) {
                second = num;
            }
            else {
                return true;
            }
        }

        return false;
    }
};
```

## Complexity

* **Time:** `O(n)` — single pass through the array
* **Space:** `O(1)` — only two variables

## Remember

```text
first  → smallest possible first element
second → smallest possible second element
num    → if num > second → triplet exists
```

**Pattern:** Maintain the smallest possible values for the first and second positions of the subsequence.


# 🍬 LeetCode 1431 — Kids With the Greatest Number of Candies

---

## 📋 Question Summary
Given an array `candies` where `candies[i]` is the number of candies the `i`-th kid has, and an integer `extraCandies` (the number of extra candies you have), determine for **each kid** whether giving them all `extraCandies` would let them have the **greatest** number of candies among all kids (ties count as greatest).

Return a boolean array `result` where `result[i]` is `true` if kid `i` could have the max after receiving the extra candies, else `false`.

**Example:**
```
candies = [2, 3, 5, 1, 3], extraCandies = 3
maxCandies = 5
result = [false, true, true, false, true]
```

---

## 🧠 Logic / Approach
1. Find `maxCandies`, the current maximum in the `candies` array (this is the bar every kid needs to reach after their boost).
2. For each kid `i`, check if `candies[i] + extraCandies >= maxCandies`.
   - If yes → that kid *could* have the greatest number of candies → `true`.
   - If no → `false`.
3. This only requires **one pass** to find the max and **one pass** to build the result → **O(n) time, O(1) extra space** (excluding output).

**Key insight:** You never need to compare kids against each other directly — just compare each kid's boosted total against the single fixed `maxCandies` value found once up front.

**Common pitfall:** Don't mutate the original `candies` array while iterating (e.g., adding `extraCandies` in place). If earlier elements get permanently boosted before later comparisons happen, the boost effectively cancels out in later checks, producing wrong results.

---

## 💻 C++ Code
```cpp
class Solution {
public:
    vector<bool> kidsWithCandies(vector<int>& candies, int extraCandies) {
        int n = candies.size();
        vector<bool> result(n, false);
        int maxCandies = *max_element(candies.begin(), candies.end());

        for (int i = 0; i < n; i++) {
            result[i] = (candies[i] + extraCandies >= maxCandies);
        }
        return result;
    }
};
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)` extra (excluding the output vector)