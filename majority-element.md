#arrays

##majority element
we consider the element as the majority element if the frequency of the element in the array is greater than the floor value of n/2

code:(this is the brute force approach)

for(value: nums){
	int freq;
	
	for(ele: nums){
		if(ele == value){
			freq++;
		}
	}
	
	if(freq > n/2){
		return ele;
	}
}

---
## products of the elements in the array except self

//this is the brute force approach
vector<int> answer
for(int i = 0; i < n; i++){
	int prod = 1;
	for(j = 0; j < n; j++){
		if(i != j){
			prod = prod * i;
		}
	ans[i] = prod;
	}
}

//optimised approach
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {

        int n = nums.size();
        vector<int> answer(n, 1);
        vector<int> prefix(n, 1);
        vector<int> suffix(n, 1);
        prefix[0] = 1;
        suffix[n - 1] = 1;

        //loop for prefix
        for(int i = 1; i < n; i++){
            prefix[i] = prefix[i - 1] * nums[i - 1];
        }

        //loop for suffix
        for(int j = n - 2; j >= 0; j--){
            suffix[j] = suffix[j + 1] * nums[j + 1];
        }

        //loop to get the answer
        for(int k = 0; k < n; k++){
            answer[k] = prefix[k] * suffix[k];
        }
        return answer;
    }
};

# C++ Pointers for LeetCode — Notes

> Quick reference for pointers/references in C++, written for someone coming from C.

---

## 1. The Basics (same as C)

```cpp
int x = 10;
int* p = &x;      // p holds address of x
cout << *p;         // dereference: 10
*p = 20;             // x is now 20
```

- `&x` → address-of
- `*p` → dereference
- `int*` → pointer type

**Syntax note:** prefer `int* p` over `int *p` (identical behavior, cleaner style).
Watch out: `int* p, q;` → `q` is a plain `int`, not a pointer, since `*` binds to the variable name, not the type. Declare pointers separately if declaring multiple.

---

## 2. References — the big new thing vs C

The most important C++-specific concept to internalize.

```cpp
int x = 10;
int& r = x;   // r is an alias for x, NOT a pointer
r = 20;        // x becomes 20
```

**Differences from pointers:**
- Must be initialized when declared; can never be "reseated" (always refers to the same variable).
- No `*` needed to access/modify — behaves exactly like the original variable.
- No such thing as a null reference (unlike pointers, which can be `nullptr`).

### Where you'll actually use this on LeetCode

Function parameters — to avoid copying and to modify the caller's data directly.

```cpp
void increment(int& x) { x++; }        // pass by reference — modifies original
void print(const vector<int>& nums) {  // pass by const reference — avoids copy, read-only
    for (int n : nums) cout << n;
}
```

Critical for recursive solutions (DFS/backtracking) — avoids copying large containers on every stack frame:

```cpp
void backtrack(vector<int>& nums, vector<int>& path, vector<vector<int>>& result) {
    // modify path in place, push to result by reference — no copies
}
```

---

## 3. `nullptr` instead of `NULL` / `0`

C++11 introduced `nullptr` — a real, typed null pointer constant.

```cpp
int* p = nullptr;
if (p == nullptr) { ... }
if (!p) { ... }   // equally idiomatic
```

---

## 4. `new` / `delete` instead of `malloc` / `free`

Common in LeetCode's linked-list and tree problems.

```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

ListNode* node = new ListNode(5);   // allocates + calls constructor
delete node;                          // frees (rarely needed on LeetCode)
```

`new` combines allocation + initialization and returns a properly-typed pointer (no cast needed, unlike `malloc`).

> On LeetCode you almost never call `delete` — the judge doesn't penalize leaks in a single run.

---

## 5. Pointer Arithmetic & Arrays — same as C

```cpp
int arr[5] = {1,2,3,4,5};
int* p = arr;        // decays to pointer to first element
cout << *(p+2);       // 3
cout << p[2];          // 3, same thing
```

Matters less on LeetCode since `vector<int>` is used instead of raw arrays most of the time, but decay rules are identical to C.

---

## 6. Pointers to Pointers / Struct Pointers

Rare to see raw `T**` on LeetCode. Much more common: **structs holding pointers to themselves** (linked lists, trees).

```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* curr = head;
    while (curr) {
        ListNode* next = curr->next;  // save next
        curr->next = prev;             // reverse the link
        prev = curr;
        curr = next;
    }
    return prev;
}
```

`->` works exactly like C: `curr->next` is shorthand for `(*curr).next`.

---

## 7. `const` Correctness

Read right-to-left:

```cpp
const int* p;        // pointer to const int — can't modify *p, CAN reassign p
int* const p;         // const pointer to int — CAN modify *p, can't reassign p
const int* const p;  // neither can change
```

Most given LeetCode function signatures use `const vector<int>&` for read-only inputs.

---

## 8. Smart Pointers (good to know, rarely needed)

C++11 RAII-style automatic memory management:

```cpp
unique_ptr<int> p = make_unique<int>(5);   // auto-deleted when p goes out of scope
shared_ptr<int> sp = make_shared<int>(5);   // reference-counted
```

Essentially never required for LeetCode — raw pointers via `new` / given struct definitions are the norm.

---

## 9. The Pattern You'll Use 90% of the Time: Tree/List Struct Pointers

Usually **provided** in the problem, not written by you:

```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};
```

The `: val(x), left(nullptr)...` is a **member initializer list** — C++ syntax for initializing fields in the constructor (more efficient than assigning inside the body).

Typical recursive traversal:

```cpp
int maxDepth(TreeNode* root) {
    if (!root) return 0;                 // null check, same as C
    return 1 + max(maxDepth(root->left), maxDepth(root->right));
}
```

---

## 10. "Two Pointers" Technique — Not Actually About Pointer Types

> ⚠️ LeetCode's "two pointer" pattern (sorted arrays, sliding window) almost always uses **integer indices**, not `int*` variables.

```cpp
bool twoSum(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;   // "pointers" — just ints here
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) return true;
        else if (sum < target) left++;
        else right--;
    }
    return false;
}
```

Don't confuse the idiom's name with real `int*` pointer arithmetic — 95% of the time "two pointers" means two indices into a vector.

---

## 11. Iterators — C++'s "Safe Pointer" for Containers

STL containers expose iterators, which behave like pointers (`*it`, `it++`) but work uniformly across container types.

```cpp
vector<int> v = {1,2,3};
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it;
}
```

On LeetCode, range-based for loops (`for (int n : v)`) are more common, but iterators show up when erasing/inserting mid-container or using `lower_bound` / `upper_bound`.

---

## Quick Cheat-Sheet: C → C++ Swaps

| C | C++ |
|---|---|
| `NULL` | `nullptr` |
| `malloc` / `free` | `new` / `delete` |
| passing pointer to avoid copy | passing by reference (`T&` or `const T&`) |
| raw array + size param | `vector<T>&` |
| `char*` strings | `string&` |
| `struct Node* p` everywhere | same, but often combined with references for tree/list algorithms |

---

## Reference Type Summary

| Syntax | Meaning |
|---|---|
| `int* p` | Pointer to int |
| `int& r` | Reference to int (alias) |
| `const int* p` | Pointer to const int (data immutable, pointer reassignable) |
| `int* const p` | Const pointer to int (pointer fixed, data mutable) |
| `const int* const p` | Both fixed |
| `const T&` (param) | Pass by reference, read-only, no copy — very common in LeetCode signatures |