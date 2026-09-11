# C++ Strings — LeetCode Cheat Sheet

## 1. Basics

```cpp
#include <string>
using namespace std;

string s = "hello";
string s2(5, 'a');        // "aaaaa"
string s3 = s + " world"; // concatenation
```

- `std::string` is mutable, dynamically sized, null-terminated internally but you don't manage `\0` yourself.
- Indexing: `s[i]` (no bounds check) vs `s.at(i)` (throws `out_of_range`).
- `s.length()` and `s.size()` are identical — use whichever reads better.

---

## 2. Core Operations

| Operation | Code | Notes |
|---|---|---|
| Length | `s.size()` / `s.length()` | O(1) |
| Access char | `s[i]` | no bounds check |
| Substring | `s.substr(pos, len)` | O(len), `len` optional (goes to end) |
| Find | `s.find("abc")` | returns index or `string::npos` |
| Find from right | `s.rfind("abc")` | last occurrence |
| Replace | `s.replace(pos, len, "new")` | in-place |
| Insert | `s.insert(pos, "abc")` | shifts rest right |
| Erase | `s.erase(pos, len)` | removes chars |
| Append | `s += "abc"` / `s.append("abc")` | |
| Compare | `s.compare(t)` or `s == t` | lexicographic |
| Reverse | `reverse(s.begin(), s.end())` | needs `<algorithm>` |
| Sort chars | `sort(s.begin(), s.end())` | needs `<algorithm>` |
| Clear | `s.clear()` | |
| Empty check | `s.empty()` | faster than `s.size()==0` |

**Always check `string::npos`:**
```cpp
if (s.find("abc") != string::npos) { /* found */ }
```

---

## 3. Building Strings Efficiently

Avoid repeated `+=` causing reallocations in tight loops when size is known:
```cpp
string result;
result.reserve(n);       // pre-allocate
for (char c : chars) result.push_back(c);
```

`push_back(char)` is O(1) amortized — prefer it over `+= char` in loops (though both are usually fine in practice).

---

## 4. Conversions

```cpp
// string <-> number
int x = stoi("123");
long y = stol("123456789");
double d = stod("3.14");
string s = to_string(456);

// char <-> int
int digit = c - '0';       // '7' - '0' = 7
char c = digit + '0';      // 7 + '0' = '7'
int lower_to_upper = c - 'a' + 'A';

// char classification (<cctype>)
isalpha(c); isdigit(c); isalnum(c);
isupper(c); islower(c); isspace(c);
toupper(c); tolower(c);
```

---

## 5. Splitting / Tokenizing

C++ has no built-in `split()`. Common patterns:

**Using `stringstream` (split by whitespace or a delimiter char):**
```cpp
#include <sstream>
stringstream ss(s);
string word;
vector<string> tokens;
while (ss >> word) tokens.push_back(word);   // splits on whitespace

// custom delimiter
while (getline(ss, word, ',')) tokens.push_back(word);
```

**Manual two-pointer split** (avoids stringstream overhead — often preferred in interviews):
```cpp
vector<string> split(const string& s, char delim) {
    vector<string> res;
    int start = 0;
    for (int i = 0; i <= (int)s.size(); i++) {
        if (i == (int)s.size() || s[i] == delim) {
            res.push_back(s.substr(start, i - start));
            start = i + 1;
        }
    }
    return res;
}
```

---

## 6. Common LeetCode Patterns

### A. Two Pointers (palindrome check, reverse, etc.)
```cpp
bool isPalindrome(string s) {
    int l = 0, r = s.size() - 1;
    while (l < r) {
        if (s[l] != s[r]) return false;
        l++; r--;
    }
    return true;
}
```

### B. Sliding Window (longest substring without repeats, etc.)
```cpp
int lengthOfLongestSubstring(string s) {
    unordered_set<char> seen;
    int l = 0, best = 0;
    for (int r = 0; r < (int)s.size(); r++) {
        while (seen.count(s[r])) {
            seen.erase(s[l]);
            l++;
        }
        seen.insert(s[r]);
        best = max(best, r - l + 1);
    }
    return best;
}
```

### C. Frequency Counting (anagrams, char counts)
```cpp
bool isAnagram(string s, string t) {
    if (s.size() != t.size()) return false;
    int count[26] = {0};
    for (char c : s) count[c - 'a']++;
    for (char c : t) count[c - 'a']--;
    for (int c : count) if (c != 0) return false;
    return true;
}
```

### D. Hashing with `unordered_map<string, ...>`
```cpp
// Group Anagrams
vector<vector<string>> groupAnagrams(vector<string>& strs) {
    unordered_map<string, vector<string>> groups;
    for (string& s : strs) {
        string key = s;
        sort(key.begin(), key.end());
        groups[key].push_back(s);
    }
    vector<vector<string>> res;
    for (auto& [k, v] : groups) res.push_back(v);
    return res;
}
```

### E. Building/Backtracking on Strings
```cpp
void backtrack(string& current, vector<string>& result, /* args */) {
    if (/* base case */) {
        result.push_back(current);
        return;
    }
    for (/* choices */) {
        current.push_back(/* char */);
        backtrack(current, result, /* args */);
        current.pop_back();   // undo
    }
}
```

### F. String Matching (KMP-lite / brute force substring search)
```cpp
// s.find(pattern) handles most LeetCode needs directly.
// For custom implementations (e.g., "Implement strStr()"):
int strStr(string haystack, string needle) {
    int n = haystack.size(), m = needle.size();
    for (int i = 0; i + m <= n; i++) {
        if (haystack.substr(i, m) == needle) return i;
    }
    return -1;
}
```

---

## 7. Gotchas & Tips

- **`s.substr(pos, len)`** — if `len` goes past the end, it just returns up to the end (no crash). `pos` out of range throws `out_of_range`.
- **Comparing strings and chars**: `'a' < 'b'` works naturally (ASCII order) — useful for sorting.
- **Mutation while iterating**: modifying `s` (e.g., via `erase`/`insert`) while looping by index can shift subsequent indices — recheck loop bounds.
- **Pass by reference** (`string&` or `const string&`) in function params to avoid expensive copies.
- **`s.data()` / `s.c_str()`**: get raw `char*` (useful for C APIs); `c_str()` guarantees null-termination.
- **Immutable-looking ops that copy**: `+`, `substr()` create new strings — O(n) each; watch for O(n²) blowup in loops.
- **`string::npos`** is `size_t(-1)` — a huge unsigned number, never compare with `<0`.
- **Unicode/multibyte**: LeetCode strings are usually ASCII/lowercase — `char` arithmetic (`c - 'a'`) is safe in that context only.

---

## 8. Quick Reference: Useful `<algorithm>` Functions

```cpp
reverse(s.begin(), s.end());
sort(s.begin(), s.end());
transform(s.begin(), s.end(), s.begin(), ::tolower);   // lowercase whole string
count(s.begin(), s.end(), 'a');                        // count occurrences
max_element(s.begin(), s.end());
unique(s.begin(), s.end());                            // removes consecutive dupes (needs erase after)
```

---

## 9. Practice Problem Categories to Map These To

- **Two pointers**: Valid Palindrome, Reverse String, Container With Most Water (numeric but same pattern)
- **Sliding window**: Longest Substring Without Repeating Characters, Minimum Window Substring
- **Hash map / frequency**: Valid Anagram, Group Anagrams, First Unique Character
- **Backtracking**: Letter Combinations of a Phone Number, Generate Parentheses, Palindrome Partitioning
- **DP on strings**: Longest Common Subsequence, Edit Distance, Longest Palindromic Substring
- **Stack-based**: Valid Parentheses, Decode String, Basic Calculator