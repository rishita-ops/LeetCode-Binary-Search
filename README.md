# LeetCode — Binary Search

A focused Python implementation of **LeetCode's Binary Search (Easy)** that **finds a target in a sorted array in O(log n) time** by repeatedly halving the search range. Clean, minimal, and correct — this solution is the entry point to understanding the search-space-halving idea, the `left <= right` loop invariant, and the pointer updates (`mid + 1` / `mid - 1`) that every other binary search problem is built on. Binary search is one of the most reused patterns in array, math, and "search on the answer" problems.

🔗 **Problem link:** [leetcode.com/problems/binary-search](https://leetcode.com/problems/binary-search/)

---

## Problem Statement

Given an array of integers `nums` which is sorted in **ascending order**, and an integer `target`, write a function to search `target` in `nums`. If `target` exists, return its index. Otherwise, return `-1`.

You must write an algorithm with **O(log n)** runtime complexity.

**Example 1:**

```
Input:  nums = [-1,0,3,5,9,12], target = 9
Output: 4
Explanation: 9 exists in nums and its index is 4
```

**Example 2:**

```
Input:  nums = [-1,0,3,5,9,12], target = 2
Output: -1
Explanation: 2 does not exist in nums so return -1
```

**Official Constraints:**

- `1 <= nums.length <= 10^4`
- `-10^4 < nums[i], target < 10^4`
- All the integers in `nums` are **unique**
- `nums` is sorted in **ascending order**

---

## The Code

```python
#Binary Search :
#Given an array of integers nums which is sorted in ascending order, and an integer target, write a function to search target in nums.
#If target exists, then return its index. Otherwise, return -1.
#You must write an algorithm with O(log n) runtime complexity.
#donot use vectors

def binary_search(nums, target):
    left = 0
    right = len(nums) - 1

    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1

nums = [-1, 0, 3, 5, 9, 12]
target = 9
result = binary_search(nums, target)
print(result)
```

---

## The Critical Idea — Sorted Order Lets You Throw Away Half the Array

This is the most important thing to understand about this solution.

Because `nums` is sorted, one comparison with the **middle** element tells you which half the target _cannot_ be in. That half is discarded, and the search continues on the other half. The array is never modified — only the two pointers `left` and `right` move.

```
nums:    -1   0   3   5   9   12
index:    0   1   2   3   4    5
          L       M            R        mid = 2, nums[2] = 3 < 9  → target is to the right
                      L   M    R        mid = 4, nums[4] = 9 == 9 → found, return 4
```

Each step cuts the remaining range roughly in half:

```
n → n/2 → n/4 → n/8 → ... → 1      (about log₂ n steps)
```

| Approach                      | Needs sorted input? | Comparisons (n = 10,000) | Time         |
| ----------------------------- | ------------------- | ------------------------ | ------------ |
| Linear search                 | ❌ No               | up to 10,000             | O(n)         |
| **Binary search (this code)** | ✅ Yes              | about 14                 | **O(log n)** |

The sorted-order guarantee is the whole reason this works. On an unsorted array, discarding half would throw away the answer.

---

## How the Loop Works

```python
while left <= right:
    mid = (left + right) // 2
    if nums[mid] == target:
        return mid
    elif nums[mid] < target:
        left = mid + 1
    else:
        right = mid - 1
```

- **Start:** `left = 0`, `right = len(nums) - 1` — the whole array is the search range
- **Condition:** `left <= right` — the range still contains at least one element
- **Middle:** `mid = (left + right) // 2` — integer division picks the center index
- **Found:** `nums[mid] == target` — return the index immediately
- **Too small:** `nums[mid] < target` — target is on the right, so `left = mid + 1`
- **Too big:** `nums[mid] > target` — target is on the left, so `right = mid - 1`
- **Not found:** the range becomes empty (`left > right`), so the loop ends and `-1` is returned

> **Off-by-one note:** The condition is `left <= right`, not `left < right`. With `<`, a range holding exactly one element (`left == right`) would never be checked, and a target sitting in that last position would be missed. The updates are `mid + 1` and `mid - 1` (not `mid`) because `nums[mid]` has already been checked, so it must be excluded — otherwise the loop can repeat forever.

---

## Algorithm (Pseudocode)

```
left  = 0
right = n - 1

while left <= right:
    mid = (left + right) / 2
    if nums[mid] == target:  return mid
    if nums[mid] <  target:  left  = mid + 1
    else:                    right = mid - 1

return -1
```

---

## Dry Run

**Input:** `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`

| Step | `left` | `right` | `mid` | `nums[mid]` | Comparison | Action     |
| ---- | ------ | ------- | ----- | ----------- | ---------- | ---------- |
| 1    | 0      | 5       | 2     | 3           | `3 < 9`    | `left = 3` |
| 2    | 3      | 5       | 4     | 9           | `9 == 9`   | return `4` |

**Output:** `4` ✅

**Input:** `nums = [-1, 0, 3, 5, 9, 12]`, `target = 2` (not present)

| Step | `left` | `right` | `mid` | `nums[mid]` | Comparison     | Action      |
| ---- | ------ | ------- | ----- | ----------- | -------------- | ----------- |
| 1    | 0      | 5       | 2     | 3           | `3 > 2`        | `right = 1` |
| 2    | 0      | 1       | 0     | -1          | `-1 < 2`       | `left = 1`  |
| 3    | 1      | 1       | 1     | 0           | `0 < 2`        | `left = 2`  |
| 4    | 2      | 1       | —     | —           | `left > right` | loop ends   |

**Output:** `-1` ✅

---

## Complexity Analysis

| Metric | Complexity                                                                                 |
| ------ | ------------------------------------------------------------------------------------------ |
| Time   | **O(log n)** — the search range halves every iteration, so at most about log₂ n iterations |
| Space  | **O(1)** — only `left`, `right`, and `mid`; no extra data structures                       |

---

## Edge Cases

| Scenario                          | Behavior                                                                                              |
| --------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Target is the first element       | `right` keeps shrinking until `mid = 0` — found ✅                                                    |
| Target is the last element        | `left` keeps growing until `mid = n-1` — found ✅                                                     |
| Target smaller than every element | `right` drops to `-1`, loop ends — returns `-1` ✅                                                    |
| Target larger than every element  | `left` grows to `n`, loop ends — returns `-1` ✅                                                      |
| Single element `[5]`, target `5`  | `left = right = 0`, `mid = 0` — found ✅                                                              |
| Single element `[5]`, target `3`  | Checked once, then range is empty — returns `-1` ✅                                                   |
| Target falls between two elements | Range collapses to empty — returns `-1` ✅                                                            |
| Empty array `[]`                  | `right = -1`, so `left <= right` is false at once — returns `-1` ✅ (the problem guarantees `n >= 1`) |
| Duplicate values                  | Returns _an_ index of the target, not necessarily the first ⚠️ (the problem guarantees unique values) |
| Unsorted input                    | Result is unreliable — binary search requires sorted input ⚠️                                         |

---

## Code-Quality Notes

**The "do not use vectors" comment.** `vector` is a C++ container, and Python has none. This solution uses a plain Python list and no extra data structures at all, so it meets the spirit of that note: no copies, no auxiliary arrays, O(1) extra space.

**The midpoint formula.** `(left + right) // 2` is safe in Python because integers have arbitrary precision. In C++ or Java, `left + right` can overflow `int` on very large arrays, so the portable form is:

```python
mid = left + (right - left) // 2
```

It gives the same result without ever forming a large sum, and it is the better habit to carry into other languages.

**Not a direct LeetCode submission.** LeetCode calls a method on a class, and the function here is standalone with a hardcoded test. The next section has the submittable version.

---

## The Natural Extension — LeetCode Submission Format and `bisect`

**Submittable form (what LeetCode expects):**

```python
from typing import List

class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1

        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] == target:
                return mid
            elif nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1

        return -1
```

**Built-in alternative using the standard library:**

```python
from bisect import bisect_left

def search(nums, target):
    i = bisect_left(nums, target)
    if i < len(nums) and nums[i] == target:
        return i
    return -1
```

`bisect_left` returns the position where `target` would be inserted to keep the list sorted. If the element at that position equals `target`, it was found.

| Property                  | Hand-written (this code) | `bisect_left`                           |
| ------------------------- | ------------------------ | --------------------------------------- |
| Time                      | O(log n)                 | O(log n)                                |
| Space                     | O(1)                     | O(1)                                    |
| Shows the algorithm       | ✅ Yes                   | ❌ Hidden inside the library            |
| Interview-friendly        | ✅ Yes                   | ⚠️ Usually expected to write it by hand |
| Finds first of duplicates | ❌ Not guaranteed        | ✅ Yes (leftmost position)              |

---

## Repository Structure

```
LeetCode-Binary-Search/
│
├── binary_search.py      # Main Python implementation
└── README.md             # Project documentation
```

---

## How to Compile and Run

**Prerequisites:** Python 3.8+

```bash
# Clone the repository
git clone https://github.com/rishita-ops/LeetCode-Binary-Search.git
cd LeetCode-Binary-Search

# Run
python3 binary_search.py
```

**On Windows:**

```bash
python binary_search.py
```

**Expected output:**

```
4
```

---

## Key Concepts Covered

- **Search-space halving** — one comparison discards half of the remaining elements
- **Sorted input requirement** — the entire algorithm depends on ascending order
- **Two-pointer range (`left`, `right`)** — the pointers bound the region that could still contain the target
- **`left <= right` loop condition** — keeps single-element ranges in play
- **`mid + 1` / `mid - 1` updates** — exclude the element just checked, guaranteeing progress
- **Safe midpoint** — `left + (right - left) // 2` avoids overflow in C++ / Java
- **O(log n) time, O(1) space** — the efficiency target the problem requires
- **`bisect` module** — Python's built-in binary search tools

---

## Why This Problem Matters in DSA

The halve-the-range pattern appears across a broad family of problems:

| Problem / Concept                                                          | Connection                                                   |
| -------------------------------------------------------------------------- | ------------------------------------------------------------ |
| **Search Insert Position** (LeetCode #35)                                  | Same loop, but return `left` when the target is missing      |
| **First Bad Version** (LeetCode #278)                                      | Binary search on a true/false boundary instead of a value    |
| **Find First and Last Position of Element in Sorted Array** (LeetCode #34) | Two binary searches to handle duplicates                     |
| **Search in Rotated Sorted Array** (LeetCode #33)                          | Decide which half is sorted, then halve                      |
| **Sqrt(x)** (LeetCode #69)                                                 | Binary search over possible answers instead of array indices |
| **Peak Index in a Mountain Array** (LeetCode #852)                         | Use the slope at `mid` to choose a direction                 |
| **Koko Eating Bananas** (LeetCode #875)                                    | "Binary search on the answer" with a feasibility check       |
| **Guess Number Higher or Lower** (LeetCode #374)                           | Binary search driven by an external comparison function      |

Every problem that searches a sorted structure, or a monotonic yes/no condition, traces back to the halving idea introduced here.

---

## Contributing

Contributions are welcome. Consider adding:

- The **`class Solution` submission** as a separate, submittable file
- A **recursive** version of binary search for comparison
- A version that returns the **first occurrence** when duplicates exist
- A version using `bisect_left` from the standard library
- Input validation for unsorted arrays
- Implementations in C++, Java, or JavaScript

```bash
git checkout -b feature/your-feature
git commit -m "Add: your feature description"
git push origin feature/your-feature
# Then open a Pull Request
```

---

## License

This project is open-source and available under the [MIT License](LICENSE).

---

_Part of a structured LeetCode practice series — fundamentals, done right._
