---
title: Binary Search In Sorted Rotated Array A Practical Guide
date: 2025-06-18T02:04:08.681Z
description: "Master the art of performing binary search on a sorted array that has been rotated. This detailed guide covers the logic, provides a working Python example, and walks through common scenarios and edge cases for optimal understanding."
tags: [Algorithms, Binary Search, Interview Prep, Data Structures, Python]
categories: [Programming, Algorithms]
comments: true
---

Binary search is a fundamental algorithm, celebrated for its `O(log N)` efficiency when searching in sorted arrays. But what happens when that perfectly sorted array gets, well, *rotated*? This seemingly small twist can turn a straightforward problem into a mind-bender.

Today, we're diving deep into the "Search in Sorted Rotated Array" problem. It's a classic interview question and a great way to truly understand how to adapt algorithms to trickier constraints.

## What's a Sorted Rotated Array?

Imagine you have a sorted array: `[0, 1, 2, 4, 5, 6, 7]`.
Now, imagine this array is picked up from an arbitrary pivot point and the first part is moved to the end. For example, if we rotate it at the pivot `4`:

`[4, 5, 6, 7, 0, 1, 2]`

Notice a few things:
1.  It's no longer fully sorted from start to end.
2.  It *does* consist of two sorted sub-arrays: `[4, 5, 6, 7]` and `[0, 1, 2]`.
3.  The "rotation point" (or "pivot") is where the smallest element `0` appears after the largest element `7`.

The challenge is to find a target value in this array using binary search's `O(log N)` efficiency, even though the standard binary search assumption (globally sorted data) is broken.

## Why Standard Binary Search Fails

Let's say we want to find `0` in `[4, 5, 6, 7, 0, 1, 2]`.
A standard binary search would:
*   Set `low = 0`, `high = 6`.
*   `mid = 3` (element `7`).
*   Since `target (0) < nums[mid] (7)`, it would try to search the left half `[4, 5, 6]`.
*   Oops! `0` is actually in the right half of the *original* array, which is now the right half of the *rotated* array.

The problem is that the `nums[mid] < target` or `nums[mid] > target` comparison no longer reliably tells us which side of `mid` the target is on *globally*.

## The Core Idea: Identify the Sorted Half

The trick is to leverage the fact that *one half* of the array (either `low` to `mid` or `mid` to `high`) will *always* be sorted. We can use this property to narrow down our search space.

Here's the logic:

1.  **Initialize Pointers**: Set `low = 0` and `high = len(nums) - 1`.
2.  **Loop**: Continue while `low <= high`.
3.  **Calculate Mid**: Find `mid = low + (high - low) // 2`.
4.  **Target Found**: If `nums[mid] == target`, we're done! Return `mid`.
5.  **Identify Sorted Half**: This is the crucial step.
    *   **Case 1: Left half is sorted (`nums[low] <= nums[mid]`)**
        *   This means the elements from `nums[low]` to `nums[mid]` are in increasing order.
        *   Now, check if our `target` lies within this sorted range: `nums[low] <= target < nums[mid]`.
            *   If yes, the target must be in the left half. So, `high = mid - 1`.
            *   If no, the target must be in the *unsorted* right half. So, `low = mid + 1`.
    *   **Case 2: Right half is sorted (`nums[mid] < nums[low]`)**
        *   This means the elements from `nums[mid]` to `nums[high]` are in increasing order (because the pivot is in the left half).
        *   Now, check if our `target` lies within this sorted range: `nums[mid] < target <= nums[high]`.
            *   If yes, the target must be in the right half. So, `low = mid + 1`.
            *   If no, the target must be in the *unsorted* left half. So, `high = mid - 1`.
6.  **Target Not Found**: If the loop finishes (i.e., `low > high`), the target was not found in the array. Return `-1`.

Note: The comparison `nums[low] <= nums[mid]` handles the case where `low == mid`, which means the "left half" is a single element. This is correct as a single element is always "sorted." Also, for `nums[mid] < nums[low]`, it correctly identifies that the pivot must be in the left part of `[low...mid]`, making the right side `[mid...high]` sorted.

## Python Code Example

Let's put this logic into a Python function.

```python
import math

def search_rotated_array(nums: list[int], target: int) -> int:
    """
    Performs binary search on a sorted and rotated array.

    Args:
        nums: A list of integers, sorted and rotated at an unknown pivot.
              Assumes unique elements.
        target: The integer to search for.

    Returns:
        The index of the target if found, otherwise -1.
    """
    if not nums:
        return -1

    low, high = 0, len(nums) - 1

    while low <= high:
        mid = low + (high - low) // 2 # Avoids potential overflow with (low + high) // 2

        if nums[mid] == target:
            return mid

        # Determine which half is sorted
        # Case 1: Left half is sorted (nums[low] to nums[mid])
        if nums[low] <= nums[mid]:
            # Check if target is in the sorted left half
            if nums[low] <= target < nums[mid]:
                high = mid - 1 # Target is in the left sorted portion
            else:
                low = mid + 1  # Target is in the unsorted right portion
        
        # Case 2: Right half is sorted (nums[mid] to nums[high])
        else: # nums[mid] < nums[low], meaning the rotation point is in the left half
            # Check if target is in the sorted right half
            if nums[mid] < target <= nums[high]:
                low = mid + 1  # Target is in the right sorted portion
            else:
                high = mid - 1 # Target is in the unsorted left portion
    
    return -1 # Target not found

```

## Sample Runs and Outputs

Let's test our function with various scenarios.

```python
# Test cases
print(f"Searching in [4, 5, 6, 7, 0, 1, 2] for 0: {search_rotated_array([4, 5, 6, 7, 0, 1, 2], 0)}")
print(f"Searching in [4, 5, 6, 7, 0, 1, 2] for 3: {search_rotated_array([4, 5, 6, 7, 0, 1, 2], 3)}")
print(f"Searching in [4, 5, 6, 7, 0, 1, 2] for 7: {search_rotated_array([4, 5, 6, 7, 0, 1, 2], 7)}")
print(f"Searching in [4, 5, 6, 7, 0, 1, 2] for 4: {search_rotated_array([4, 5, 6, 7, 0, 1, 2], 4)}")
print(f"Searching in [1] for 1: {search_rotated_array([1], 1)}")
print(f"Searching in [1] for 0: {search_rotated_array([1], 0)}")
print(f"Searching in [] for 5: {search_rotated_array([], 5)}")
print(f"Searching in [1, 2, 3, 4, 5] (no rotation) for 3: {search_rotated_array([1, 2, 3, 4, 5], 3)}")
print(f"Searching in [5, 1, 3] for 5: {search_rotated_array([5, 1, 3], 5)}")
print(f"Searching in [5, 1, 3] for 1: {search_rotated_array([5, 1, 3], 1)}")
print(f"Searching in [5, 1, 3] for 3: {search_rotated_array([5, 1, 3], 3)}")
```

```output
Searching in [4, 5, 6, 7, 0, 1, 2] for 0: 4
Searching in [4, 5, 6, 7, 0, 1, 2] for 3: -1
Searching in [4, 5, 6, 7, 0, 1, 2] for 7: 3
Searching in [4, 5, 6, 7, 0, 1, 2] for 4: 0
Searching in [1] for 1: 0
Searching in [1] for 0: -1
Searching in [] for 5: -1
Searching in [1, 2, 3, 4, 5] (no rotation) for 3: 2
Searching in [5, 1, 3] for 5: 0
Searching in [5, 1, 3] for 1: 1
Searching in [5, 1, 3] for 3: 2
```

All test cases pass as expected!

## Detailed Walkthrough: `[4, 5, 6, 7, 0, 1, 2]`, target = `0`

Let's trace the execution step-by-step for the classic example:

`nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 0`

1.  **Initial**: `low = 0`, `high = 6`
    *   `mid = 0 + (6 - 0) // 2 = 3`
    *   `nums[mid]` (`nums[3]`) is `7`. `target (0) != 7`.
    *   **Check sorted half**: `nums[low] (4) <= nums[mid] (7)`? Yes, `[4, 5, 6, 7]` is sorted.
    *   **Check target in sorted half**: `nums[low] (4) <= target (0) < nums[mid] (7)`? No, `0` is not in `[4, 7)`.
    *   **Decision**: Target is in the *unsorted* right half. `low = mid + 1 = 3 + 1 = 4`.
    *   Current state: `low = 4`, `high = 6` (`[0, 1, 2]`)

2.  **Loop 2**: `low (4) <= high (6)`? Yes.
    *   `mid = 4 + (6 - 4) // 2 = 5`
    *   `nums[mid]` (`nums[5]`) is `1`. `target (0) != 1`.
    *   **Check sorted half**: `nums[low] (0) <= nums[mid] (1)`? Yes, `[0, 1]` is sorted.
    *   **Check target in sorted half**: `nums[low] (0) <= target (0) < nums[mid] (1)`? Yes, `0` is in `[0, 1)`.
    *   **Decision**: Target is in the sorted left half. `high = mid - 1 = 5 - 1 = 4`.
    *   Current state: `low = 4`, `high = 4` (`[0]`)

3.  **Loop 3**: `low (4) <= high (4)`? Yes.
    *   `mid = 4 + (4 - 4) // 2 = 4`
    *   `nums[mid]` (`nums[4]`) is `0`. `target (0) == 0`.
    *   **Decision**: Target found! Return `mid = 4`.

This trace perfectly shows how the algorithm efficiently narrows down the search space by always identifying and using the sorted portion of the array.

## Edge Cases and Considerations

*   **Empty Array**: The `if not nums: return -1` check handles this gracefully.
*   **Single Element Array**: Our logic handles this too. `low`, `mid`, `high` will all point to the same index, and `nums[mid] == target` will resolve it.
*   **No Rotation**: If the array is `[1, 2, 3, 4, 5]`, it effectively acts as a standard sorted array. `nums[low] <= nums[mid]` will always be true, and the target will be searched in the correct half.
*   **Target Not Present**: If the `while` loop finishes (`low > high`), it means the search space has collapsed without finding the target, and `-1` is returned.
*   **Duplicate Elements**: Note: The standard "Search in Rotated Sorted Array" problem (like LeetCode 33) usually assumes unique elements. If duplicates are allowed (e.g., LeetCode 81), the `nums[low] <= nums[mid]` condition can become ambiguous when `nums[low] == nums[mid] == nums[high]`. In such cases, you might need to increment `low` and decrement `high` to shrink the window, as the elements at `low` and `high` might be identical to `mid` and thus provide no information. This adds a slight `O(N)` worst-case complexity to what is usually `O(log N)`. Our current solution, however, is robust for unique elements.

## Time and Space Complexity

*   **Time Complexity**: `O(log N)`
    *   In each iteration of the `while` loop, we effectively reduce the search space by half, just like a standard binary search. This logarithmic reduction in search space leads to `O(log N)` time complexity.
*   **Space Complexity**: `O(1)`
    *   We only use a few constant extra variables (`low`, `high`, `mid`). No additional data structures are allocated based on the input size.

## Conclusion

The "Binary Search in Sorted Rotated Array" problem is more than just an interview hurdle; it's an excellent exercise in adapting fundamental algorithms to non-ideal conditions. By understanding how to identify and leverage the *locally* sorted segments of the array, we can maintain the efficiency benefits of binary search.

This pattern of finding a sorted subarray and then deciding which side to eliminate is a powerful technique that extends beyond this specific problem. Master it, and you'll find yourself much better equipped to tackle more complex algorithmic challenges.