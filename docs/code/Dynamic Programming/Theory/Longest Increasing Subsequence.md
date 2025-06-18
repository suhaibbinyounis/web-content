---
title: Demystifying the Longest Increasing Subsequence (LIS) Problem
date: 2025-06-18T02:04:08.681Z
description: A deep dive into the Longest Increasing Subsequence (LIS) problem, exploring brute-force, dynamic programming (O(n^2)), and optimized O(n log n) solutions with practical Python examples.
tags: ["Algorithms", "Dynamic Programming", "Binary Search", "Python", "Problem Solving", "Computer Science"]
categories: ["Computer Science", "Algorithms"]
comments: true
---

The "Longest Increasing Subsequence" (LIS) is a classic problem in computer science, often encountered in algorithm courses and competitive programming. Beyond academic interest, it has practical applications in diverse fields like bioinformatics (genomic sequence analysis), stock market analysis, and data compression.

This post will break down the LIS problem from its most naive approach to the highly optimized solution, showing you how each step improves efficiency. We'll provide clear explanations and runnable Python code examples for every concept.

Let's dive in!

## What is the Longest Increasing Subsequence (LIS)?

Given an unsorted array of integers, the Longest Increasing Subsequence (LIS) is the longest possible subsequence of numbers in that array such that all elements in the subsequence are in increasing order.

**Key Terms:**
*   **Subsequence:** A sequence that can be derived from another sequence by deleting some or no elements without changing the order of the remaining elements. For example, `[1, 3, 5]` is a subsequence of `[1, 2, 3, 4, 5]`.
*   **Increasing:** Each element is strictly greater than the previous one.

**Example:**

Consider the array `[10, 22, 9, 33, 21, 50, 41, 60]`.

Some increasing subsequences are:
*   `[10, 22, 33, 50, 60]` (Length: 5)
*   `[10, 21, 41, 60]` (Length: 4)
*   `[9, 21, 41, 60]` (Length: 4)
*   `[10, 22, 33]` (Length: 3)

The **Longest Increasing Subsequence** in this example is `[10, 22, 33, 50, 60]`, with a length of 5.

Note: There can be multiple LISs of the same maximum length. For instance, if `[1, 3, 2, 4]` were the input, `[1, 3, 4]` and `[1, 2, 4]` are both LISs of length 3. The problem usually asks for the length, not the subsequence itself.

## Approach 1: Brute Force (Exponential Complexity)

The most straightforward, albeit inefficient, way to solve LIS is to generate all possible subsequences, filter out the ones that are not increasing, and then find the maximum length among the remaining increasing subsequences.

### Concept

This approach can be thought of recursively. For each element in the array, you have two choices:
1.  **Include it** in the current subsequence (if it's greater than the previous element included).
2.  **Exclude it** from the current subsequence.

By exploring all these choices, you can find all increasing subsequences and then determine the longest.

### Explanation

Let's define a recursive function `find_lis(index, prev_val)`:
*   `index`: The current element we are considering in the array.
*   `prev_val`: The value of the last element included in our current increasing subsequence. We only include `arr[index]` if `arr[index] > prev_val`.

The base case is when `index` reaches the end of the array, at which point the length of the current subsequence is 0.

### Complexity

*   **Time Complexity**: O(2^n). For each element, we essentially make two recursive calls (include or exclude). This leads to an exponential growth in computations.
*   **Space Complexity**: O(n) for the recursion stack.

Due to its exponential nature, this approach is impractical for even moderately sized inputs (e.g., n > 20).

### Code Example (Python)

```python
def find_lis_brute_force(nums):
    n = len(nums)

    # Helper recursive function
    # current_index: index of the current element being considered
    # prev_index: index of the last element included in the LIS so far
    def solve(current_index, prev_index):
        # Base case: if we've gone through all elements
        if current_index == n:
            return 0

        # Option 1: Exclude the current element
        # We don't include nums[current_index], so we move to the next element
        # without changing prev_index
        length_excluding_current = solve(current_index + 1, prev_index)

        # Option 2: Include the current element (if it's greater than the previous)
        length_including_current = 0
        if prev_index == -1 or nums[current_index] > nums[prev_index]:
            # If prev_index is -1, it means this is the first element of the subsequence
            # Otherwise, check if nums[current_index] is strictly greater
            length_including_current = 1 + solve(current_index + 1, current_index)
        
        # Return the maximum of both options
        return max(length_excluding_current, length_including_current)

    # Start the recursion from the first element, with no previous element (-1)
    return solve(0, -1)

print("--- Brute Force Approach ---")
# Example 1
nums1 = [10, 22, 9, 33, 21, 50, 41, 60]
print(f"Input: {nums1}")
print(f"LIS Length: {find_lis_brute_force(nums1)}")

# Example 2
nums2 = [0, 1, 0, 3, 2, 3]
print(f"Input: {nums2}")
print(f"LIS Length: {find_lis_brute_force(nums2)}")

# Example 3
nums3 = [7, 7, 7, 7, 7, 7, 7]
print(f"Input: {nums3}")
print(f"LIS Length: {find_lis_brute_force(nums3)}")
```

### Sample Output

```output
--- Brute Force Approach ---
Input: [10, 22, 9, 33, 21, 50, 41, 60]
LIS Length: 5
Input: [0, 1, 0, 3, 2, 3]
LIS Length: 4
Input: [7, 7, 7, 7, 7, 7, 7]
LIS Length: 1
```

## Approach 2: Dynamic Programming (O(n^2) Complexity)

The brute-force solution suffers from recomputing the same subproblems multiple times. This is a classic indicator that Dynamic Programming (DP) can be applied.

### Concept

Dynamic Programming solves problems by breaking them down into simpler overlapping subproblems and storing the results of these subproblems to avoid redundant calculations.

For LIS, we can define `dp[i]` as the length of the **Longest Increasing Subsequence ending at index `i`** (meaning `nums[i]` *must* be the last element of the LIS).

### Explanation

To calculate `dp[i]`:
1.  Initialize `dp[i] = 1` (because `nums[i]` itself forms an increasing subsequence of length 1).
2.  Iterate `j` from `0` to `i-1`.
3.  If `nums[i] > nums[j]` (meaning `nums[i]` can extend an increasing subsequence ending at `j`), then we can potentially form a longer LIS ending at `i`.
4.  Update `dp[i] = max(dp[i], dp[j] + 1)`. We take the maximum because there might be multiple `j`'s that could precede `i`, and we want the one that yields the longest subsequence.

After computing `dp` values for all `i` from `0` to `n-1`, the maximum value in the `dp` array will be the length of the overall LIS.

### Algorithm Steps

1.  Create a `dp` array of size `n` (where `n` is the length of `nums`).
2.  Initialize all `dp[i]` to `1`.
3.  Iterate `i` from `0` to `n-1`:
    *   For each `i`, iterate `j` from `0` to `i-1`:
        *   If `nums[i] > nums[j]`:
            *   `dp[i] = max(dp[i], dp[j] + 1)`
4.  The final result is `max(dp)` (or `0` if `nums` is empty).

### Complexity

*   **Time Complexity**: O(n^2). We have two nested loops, each iterating up to `n` times.
*   **Space Complexity**: O(n) for the `dp` array.

This is a significant improvement over the brute-force method, making it feasible for `n` up to a few thousand.

### Code Example (Python)

```python
def find_lis_dp(nums):
    n = len(nums)
    if n == 0:
        return 0

    # dp[i] will store the length of the LIS ending at index i
    dp = [1] * n

    # Fill dp[] table in bottom-up manner
    for i in range(1, n):
        for j in range(i):
            if nums[i] > nums[j]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    # The LIS length is the maximum value in dp array
    return max(dp)

print("\n--- Dynamic Programming (O(n^2)) Approach ---")
# Example 1
nums1 = [10, 22, 9, 33, 21, 50, 41, 60]
print(f"Input: {nums1}")
print(f"LIS Length: {find_lis_dp(nums1)}")

# Example 2
nums2 = [0, 1, 0, 3, 2, 3]
print(f"Input: {nums2}")
print(f"LIS Length: {find_lis_dp(nums2)}")

# Example 3
nums3 = [7, 7, 7, 7, 7, 7, 7]
print(f"Input: {nums3}")
print(f"LIS Length: {find_lis_dp(nums3)}")

# Example 4: A longer sequence to show O(N^2) still works
nums4 = [3, 10, 2, 1, 20, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]
print(f"Input: {nums4}")
print(f"LIS Length: {find_lis_dp(nums4)}") # Expected: [3, 4, 5, ..., 18] -> length 16
```

### Sample Output

```output
--- Dynamic Programming (O(n^2)) Approach ---
Input: [10, 22, 9, 33, 21, 50, 41, 60]
LIS Length: 5
Input: [0, 1, 0, 3, 2, 3]
LIS Length: 4
Input: [7, 7, 7, 7, 7, 7, 7]
LIS Length: 1
Input: [3, 10, 2, 1, 20, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]
LIS Length: 16
```

## Approach 3: Dynamic Programming with Binary Search (O(n log n) Complexity)

This is the most optimized and often preferred solution for LIS, especially in competitive programming or when `n` is large (up to 10^5 or more). It's less intuitive than the O(n^2) DP but incredibly clever.

### Concept

Instead of storing the length of LIS *ending* at each index, we maintain an array (let's call it `tails`) that stores the **smallest tail of all increasing subsequences of a certain length**.

Specifically, `tails[k]` will store the smallest ending element among all increasing subsequences of length `k+1`.

Let's walk through an example: `nums = [10, 22, 9, 33, 21, 50, 41, 60]`

1.  Initialize `tails = []`.
2.  **`num = 10`**: `tails` is empty. Add `10`. `tails = [10]` (LIS of length 1 ending with 10).
3.  **`num = 22`**: `22 > tails[-1]` (22 > 10). Extend the longest subsequence. Add `22`. `tails = [10, 22]` (LIS of length 2 ending with 22).
4.  **`num = 9`**: `9` is not greater than `tails[-1]` (9 < 22). We need to find where `9` can replace an existing tail to form a *new* increasing subsequence of the *same length* but with a *smaller ending element*. This is crucial because a smaller ending element gives future numbers more options to extend the subsequence.
    *   Binary search for `9` in `tails`. `bisect_left` will return index `0` (where `10` is).
    *   Replace `tails[0]` with `9`. `tails = [9, 22]` (LIS of length 1 ending with 9, LIS of length 2 ending with 22).
    *   Note: `[9, 22]` is not an actual LIS of `nums`, but `tails[0]` means "the smallest tail for LIS of length 1 is 9", and `tails[1]` means "the smallest tail for LIS of length 2 is 22".
5.  **`num = 33`**: `33 > tails[-1]` (33 > 22). Add `33`. `tails = [9, 22, 33]`.
6.  **`num = 21`**: `21` is not greater than `tails[-1]` (21 < 33). Binary search for `21`. `bisect_left` returns index `1` (where `22` is). Replace `tails[1]` with `21`. `tails = [9, 21, 33]`.
7.  **`num = 50`**: `50 > tails[-1]` (50 > 33). Add `50`. `tails = [9, 21, 33, 50]`.
8.  **`num = 41`**: `41` is not greater than `tails[-1]` (41 < 50). Binary search for `41`. `bisect_left` returns index `3` (where `50` is). Replace `tails[3]` with `41`. `tails = [9, 21, 33, 41]`.
9.  **`num = 60`**: `60 > tails[-1]` (60 > 41). Add `60`. `tails = [9, 21, 33, 41, 60]`.

After processing all numbers, the length of the `tails` array is the length of the LIS. In this case, `len(tails) = 5`.

**Important Note:** The `tails` array does **not** represent an actual increasing subsequence from the input array. It's a conceptual array where `tails[i]` stores the smallest possible ending number for an increasing subsequence of length `i+1`. Its purpose is to track the minimum tail for each possible LIS length, which allows for efficient updates using binary search. The *length* of `tails` at the end *is* the LIS length.

### Algorithm Steps

1.  Initialize an empty list `tails`. This list will always be sorted in increasing order.
2.  For each `num` in the input `nums` array:
    *   Use binary search (`bisect_left` in Python's `bisect` module) to find the insertion point `i` for `num` in `tails`. `bisect_left(a, x)` returns an insertion point which comes before (to the left of) any existing entries of `x` in `a`.
    *   **If `i` is equal to `len(tails)`**: This means `num` is greater than all elements currently in `tails`. Append `num` to `tails`. This extends the longest increasing subsequence found so far.
    *   **Else (`i` is less than `len(tails)`)**: This means `num` can replace `tails[i]`. Replace `tails[i]` with `num`. We are effectively finding an increasing subsequence of length `i+1` that ends with a smaller value (`num`), which is always beneficial for extending it further.
3.  The length of the `tails` list at the end is the length of the LIS.

### Complexity

*   **Time Complexity**: O(n log n). We iterate through `n` numbers, and for each number, we perform a binary search operation on the `tails` array, which takes O(log k) time, where `k` is the current length of `tails` (at most `n`). So, `n` operations * `log n` per operation = O(n log n).
*   **Space Complexity**: O(n) for the `tails` array in the worst case (e.g., a sorted array).

### Code Example (Python)

Python's `bisect` module provides efficient binary search functions. `bisect_left(a, x)` is perfect for finding the insertion point to maintain sorted order.

```python
import bisect

def find_lis_optimized(nums):
    tails = [] # This will store the smallest tail of all increasing subsequences of length i+1

    for num in nums:
        # Find the insertion point for 'num' in 'tails'
        # 'idx' will be the index where 'num' should be inserted
        # if 'num' is already in 'tails', 'idx' will be the index of the first occurrence
        idx = bisect.bisect_left(tails, num)

        if idx == len(tails):
            # If 'num' is greater than all elements in 'tails',
            # it means we found an increasing subsequence that is one element longer.
            tails.append(num)
        else:
            # If 'num' is not greater than all elements in 'tails',
            # it means 'num' can replace an existing tail to form an
            # increasing subsequence of the same length but with a smaller ending element.
            # This is beneficial because a smaller ending allows more numbers to extend the subsequence.
            tails[idx] = num
            
    # The length of 'tails' array is the length of the LIS
    return len(tails)

print("\n--- Dynamic Programming with Binary Search (O(n log n)) Approach ---")
# Example 1
nums1 = [10, 22, 9, 33, 21, 50, 41, 60]
print(f"Input: {nums1}")
print(f"LIS Length: {find_lis_optimized(nums1)}")

# Example 2
nums2 = [0, 1, 0, 3, 2, 3]
print(f"Input: {nums2}")
print(f"LIS Length: {find_lis_optimized(nums2)}")

# Example 3
nums3 = [7, 7, 7, 7, 7, 7, 7]
print(f"Input: {nums3}")
print(f"LIS Length: {find_lis_optimized(nums3)}")

# Example 4: A longer sequence
nums4 = [3, 10, 2, 1, 20, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]
print(f"Input: {nums4}")
print(f"LIS Length: {find_lis_optimized(nums4)}")

# Example 5: Another test case
nums5 = [1, 2, 4, 3, 5, 4, 7, 2]
print(f"Input: {nums5}")
print(f"LIS Length: {find_lis_optimized(nums5)}") # Expected: [1, 2, 3, 4, 7] or [1, 2, 4, 5, 7] -> length 5
```

### Sample Output

```output
--- Dynamic Programming with Binary Search (O(n log n)) Approach ---
Input: [10, 22, 9, 33, 21, 50, 41, 60]
LIS Length: 5
Input: [0, 1, 0, 3, 2, 3]
LIS Length: 4
Input: [7, 7, 7, 7, 7, 7, 7]
LIS Length: 1
Input: [3, 10, 2, 1, 20, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]
LIS Length: 16
Input: [1, 2, 4, 3, 5, 4, 7, 2]
LIS Length: 5
```

## Comparison and When to Use Which

| Approach                    | Time Complexity | Space Complexity | Best for                                              | Notes                                                              |
| :-------------------------- | :-------------- | :--------------- | :---------------------------------------------------- | :----------------------------------------------------------------- |
| **Brute Force**             | O(2^n)          | O(n)             | Pure conceptual understanding; very small `n` (< 20) | Impractical for real-world scenarios due to exponential growth.    |
| **Dynamic Programming**     | O(n^2)          | O(n)             | `n` up to ~5,000; easier to grasp intuitively       | Standard interview solution; good balance of performance and clarity. |
| **DP + Binary Search**      | O(n log n)      | O(n)             | Large `n` (up to 10^6); competitive programming      | Most efficient; crucial for time-sensitive applications.           |

**Note:** While the O(n log n) solution is optimal for finding the *length* of the LIS, reconstructing the actual LIS sequence (not just its length) often requires additional tracking or a slightly modified DP approach, which might increase complexity or space slightly. The common LIS problem typically asks only for the length.

## Practical Considerations / Extensions

*   **Reconstructing the LIS**: To reconstruct the actual LIS, you'd typically need to store not just the `dp[i]` values but also a `predecessor[i]` array, which stores the index `j` that led to `dp[i]`. After computing all `dp` values, you find the maximum `dp` value, and then backtrack using the `predecessor` array to build the sequence in reverse.
*   **Longest Non-Decreasing Subsequence**: If the requirement is "non-decreasing" (`nums[i] >= nums[j]`), simply change the comparison from `>` to `>=`.
*   **Longest Decreasing Subsequence**: This is effectively the LIS problem on the input array sorted in reverse order, or by changing the comparison from `>` to `<`.

## Conclusion

The Longest Increasing Subsequence problem is a fantastic case study in algorithm optimization. Starting from a basic exponential solution, we can see how Dynamic Programming significantly improves performance by tackling overlapping subproblems, and then how a clever application of binary search can bring it down to a near-linear time complexity.

Understanding these different approaches not only helps you solve this specific problem but also builds a strong foundation for recognizing similar patterns in other algorithmic challenges. Practice implementing all three versions to solidify your understanding. Happy coding!
