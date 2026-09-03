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
