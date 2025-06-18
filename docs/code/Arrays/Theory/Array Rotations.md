---
title: Array Rotations - A Deep Dive with Practical Examples
date: 2025-06-18T02:04:08.681Z
description: Master array rotations – left and right – with detailed explanations and Python code examples for various algorithms, from brute force to optimal O(N) solutions. Understand their trade-offs and common pitfalls.
tags: ["arrays", "algorithms", "data structures", "programming", "python", "interview-prep"]
categories: ["Computer Science", "Algorithms", "Programming"]
comments: true
---

Array rotations are a fundamental concept in computer science and a frequent topic in coding interviews. They involve shifting the elements of an array by a certain number of positions, either to the left or to the right. While seemingly simple, optimizing these operations can lead to elegant and efficient algorithms.

In this post, we'll break down array rotations, explore various methods to perform them, and analyze their time and space complexity. We'll provide clear explanations and practical Python examples for each.

## What is an Array Rotation?

Imagine an array of numbers. A "rotation" means taking elements from one end and re-inserting them at the other end, without changing the relative order of the *remaining* elements.

There are two primary types of rotations:

1.  **Left Rotation**: Elements are shifted towards the beginning of the array. The element at the first position moves to the last position.
2.  **Right Rotation**: Elements are shifted towards the end of the array. The element at the last position moves to the first position.

Let's use an example:
Original Array: `[1, 2, 3, 4, 5]`

*   **Left rotation by 1**: `[2, 3, 4, 5, 1]`
*   **Right rotation by 1**: `[5, 1, 2, 3, 4]`

The number of rotations `d` can be any non-negative integer. If `d` is greater than the array's length `n`, the effective rotations will be `d % n`. For example, rotating an array of 5 elements by 7 positions left is the same as rotating it by `7 % 5 = 2` positions left.

## Left Rotation

We'll start with left rotations, as the concepts for right rotations can often be derived from them.

### 1. Naive Approach: One by One Rotation

This is the simplest to understand but often the least efficient for large `d`. You rotate the array by one position `d` times.

**How it works**:
1.  Store the first element in a temporary variable.
2.  Shift all remaining `n-1` elements one position to the left.
3.  Place the temporary element at the last position.
4.  Repeat this process `d` times.

**Time Complexity**: O(n*d) - For each of `d` rotations, you iterate through `n-1` elements.
**Space Complexity**: O(1) - Only a single temporary variable is used.

**Example (Python):**

```python
def left_rotate_one_by_one(arr, d):
    n = len(arr)
    if n == 0 or d == 0:
        return arr

    # Handle cases where d is greater than n
    d = d % n

    for _ in range(d):
        temp = arr[0]
        for i in range(n - 1):
            arr[i] = arr[i + 1]
        arr[n - 1] = temp
    return arr

# Test cases
print("One-by-one rotation:")
my_array = [1, 2, 3, 4, 5, 6, 7]
rotations = 3
print(f"Original: {my_array.copy()}, Rotations: {rotations}")
print(f"Rotated: {left_rotate_one_by_one(my_array.copy(), rotations)}")

my_array_2 = [10, 20, 30]
rotations_2 = 1
print(f"Original: {my_array_2.copy()}, Rotations: {rotations_2}")
print(f"Rotated: {left_rotate_one_by_one(my_array_2.copy(), rotations_2)}")

my_array_3 = [1, 2, 3, 4, 5]
rotations_3 = 5 # Rotate by n positions
print(f"Original: {my_array_3.copy()}, Rotations: {rotations_3}")
print(f"Rotated: {left_rotate_one_by_one(my_array_3.copy(), rotations_3)}")
```

```output
One-by-one rotation:
Original: [1, 2, 3, 4, 5, 6, 7], Rotations: 3
Rotated: [4, 5, 6, 7, 1, 2, 3]
Original: [10, 20, 30], Rotations: 1
Rotated: [20, 30, 10]
Original: [1, 2, 3, 4, 5], Rotations: 5
Rotated: [1, 2, 3, 4, 5]
```

### 2. Using a Temporary Array (or Python Slicing)

This method is generally more efficient for larger `d` values compared to the naive approach.

**How it works**:
1.  Store the first `d` elements of the array in a temporary array.
2.  Shift the remaining `n-d` elements to the beginning of the original array.
3.  Copy the elements from the temporary array to the end of the original array.

**Time Complexity**: O(n) - You iterate through the array roughly three times (copy `d` elements, shift `n-d` elements, copy `d` elements back). Each operation is proportional to `n`.
**Space Complexity**: O(d) - We need a temporary array to store `d` elements. In Python, slicing might create a new array of size `n`, making it O(n) in the worst case for memory, but for the conceptual approach, it's O(d).

**Example (Python):**

```python
def left_rotate_temp_array(arr, d):
    n = len(arr)
    if n == 0 or d == 0:
        return arr

    d = d % n # Normalize d

    # Pythonic way using slicing (creates new lists)
    # rotated_arr = arr[d:] + arr[:d]
    # return rotated_arr

    # In-place modification (more aligned with classic array problems)
    # Note: This is still O(n) time, O(d) or O(n) space for explicit temp_arr,
    # but aims to modify the original list if it were a fixed-size array.
    temp = []
    for i in range(d):
        temp.append(arr[i])
    
    # Shift remaining elements
    for i in range(n - d):
        arr[i] = arr[i + d]
    
    # Copy temp elements back
    for i in range(d):
        arr[n - d + i] = temp[i]
    
    return arr

# Test cases
print("\nTemporary array rotation:")
my_array = [1, 2, 3, 4, 5, 6, 7]
rotations = 3
print(f"Original: {my_array.copy()}, Rotations: {rotations}")
print(f"Rotated: {left_rotate_temp_array(my_array.copy(), rotations)}")

my_array_2 = [10, 20, 30, 40, 50]
rotations_2 = 2
print(f"Original: {my_array_2.copy()}, Rotations: {rotations_2}")
print(f"Rotated: {left_rotate_temp_array(my_array_2.copy(), rotations_2)}")
```

```output
Temporary array rotation:
Original: [1, 2, 3, 4, 5, 6, 7], Rotations: 3
Rotated: [4, 5, 6, 7, 1, 2, 3]
Original: [10, 20, 30, 40, 50], Rotations: 2
Rotated: [30, 40, 50, 10, 20]
```

### 3. Juggling Algorithm (GCD-based)

This is an optimal in-place solution for left rotation. It's a bit more complex but very efficient.

**How it works**:
The key idea is that instead of moving elements one by one or using a temporary array, we can move elements in cycles. The number of such cycles is equal to the Greatest Common Divisor (GCD) of `n` (array length) and `d` (number of rotations).

For each cycle:
1.  Start at an index `i` (from `0` to `GCD(n, d) - 1`).
2.  Store the element `arr[i]` in a temporary variable.
3.  Move `arr[i + d]`, `arr[i + 2*d]`, ... into the current position, until you wrap around and come back to the starting point of the cycle.
4.  Place the stored temporary element in the final position of the cycle.

**Example walk-through (for `arr = [1, 2, 3, 4, 5, 6, 7]`, `d = 3`):**
`n = 7`, `d = 3`. `GCD(7, 3) = 1`. This means there will be only one cycle.
*   Start `i = 0`. `temp = arr[0] = 1`.
*   `j = 0`.
*   `k = (j + d) % n = (0 + 3) % 7 = 3`. `arr[0] = arr[3] = 4`. Array: `[4, 2, 3, 4, 5, 6, 7]`
*   `j = 3`.
*   `k = (j + d) % n = (3 + 3) % 7 = 6`. `arr[3] = arr[6] = 7`. Array: `[4, 2, 3, 7, 5, 6, 7]`
*   `j = 6`.
*   `k = (j + d) % n = (6 + 3) % 7 = 9 % 7 = 2`. `arr[6] = arr[2] = 3`. Array: `[4, 2, 3, 7, 5, 6, 3]`
*   `j = 2`.
*   `k = (j + d) % n = (2 + 3) % 7 = 5`. `arr[2] = arr[5] = 6`. Array: `[4, 2, 6, 7, 5, 6, 3]`
*   `j = 5`.
*   `k = (j + d) % n = (5 + 3) % 7 = 8 % 7 = 1`. `arr[5] = arr[1] = 2`. Array: `[4, 2, 6, 7, 5, 2, 3]`
*   `j = 1`.
*   `k = (j + d) % n = (1 + 3) % 7 = 4`. `arr[1] = arr[4] = 5`. Array: `[4, 5, 6, 7, 5, 2, 3]`
*   `j = 4`.
*   `k = (j + d) % n = (4 + 3) % 7 = 0`. `arr[4] = temp = 1`. Array: `[4, 5, 6, 7, 1, 2, 3]`

The loop terminates when `k` becomes `i` (the starting index of the current cycle).

**Time Complexity**: O(n) - Each element is visited and moved exactly once.
**Space Complexity**: O(1) - Only a few temporary variables are used.

**Example (Python):**

```python
import math

def left_rotate_juggling(arr, d):
    n = len(arr)
    if n == 0 or d == 0:
        return arr

    d = d % n # Normalize d

    g_c_d = math.gcd(n, d)

    for i in range(g_c_d):
        # Move elements of current cycle
        temp = arr[i]
        j = i
        while True:
            k = (j + d) % n
            if k == i:
                break
            arr[j] = arr[k]
            j = k
        arr[j] = temp
    return arr

# Test cases
print("\nJuggling Algorithm rotation:")
my_array = [1, 2, 3, 4, 5, 6, 7]
rotations = 3
print(f"Original: {my_array.copy()}, Rotations: {rotations}")
print(f"Rotated: {left_rotate_juggling(my_array.copy(), rotations)}")

my_array_2 = [1, 2, 3, 4, 5, 6]
rotations_2 = 2 # GCD(6, 2) = 2, so two cycles
print(f"Original: {my_array_2.copy()}, Rotations: {rotations_2}")
print(f"Rotated: {left_rotate_juggling(my_array_2.copy(), rotations_2)}")

my_array_3 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
rotations_3 = 3 # GCD(12, 3) = 3, so three cycles
print(f"Original: {my_array_3.copy()}, Rotations: {rotations_3}")
print(f"Rotated: {left_rotate_juggling(my_array_3.copy(), rotations_3)}")
```

```output
Juggling Algorithm rotation:
Original: [1, 2, 3, 4, 5, 6, 7], Rotations: 3
Rotated: [4, 5, 6, 7, 1, 2, 3]
Original: [1, 2, 3, 4, 5, 6], Rotations: 2
Rotated: [3, 4, 5, 6, 1, 2]
Original: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12], Rotations: 3
Rotated: [4, 5, 6, 7, 8, 9, 10, 11, 12, 1, 2, 3]
```

### 4. Reversal Algorithm

This is another elegant and highly efficient in-place method for array rotations.

**How it works**:
The intuition comes from the fact that a left rotation by `d` splits the array into two parts: `A = arr[0...d-1]` and `B = arr[d...n-1]`. The rotation results in `BA`.
To achieve `BA` from `AB` using reversals:
1.  Reverse the first part `A`: `A^R B`
2.  Reverse the second part `B`: `A^R B^R`
3.  Reverse the entire array: `(A^R B^R)^R = B^(R R) A^(R R) = BA`

**Example walk-through (for `arr = [1, 2, 3, 4, 5, 6, 7]`, `d = 3`):**
Original: `[1, 2, 3 | 4, 5, 6, 7]` (A is `[1, 2, 3]`, B is `[4, 5, 6, 7]`)

1.  Reverse A (`arr[0...d-1]`):
    `[3, 2, 1 | 4, 5, 6, 7]`

2.  Reverse B (`arr[d...n-1]`):
    `[3, 2, 1 | 7, 6, 5, 4]`

3.  Reverse the entire array (`arr[0...n-1]`):
    `[4, 5, 6, 7, 1, 2, 3]`

**Time Complexity**: O(n) - Each reversal takes O(length_of_segment), so overall it's O(n).
**Space Complexity**: O(1) - All operations are in-place.

**Example (Python):**

```python
def _reverse_subarray(arr, start, end):
    while start < end:
        arr[start], arr[end] = arr[end], arr[start]
        start += 1
        end -= 1

def left_rotate_reversal(arr, d):
    n = len(arr)
    if n == 0 or d == 0:
        return arr

    d = d % n # Normalize d

    # Handle edge case where d is 0 or n (no rotation needed)
    if d == 0:
        return arr

    # 1. Reverse the first 'd' elements
    _reverse_subarray(arr, 0, d - 1)
    
    # 2. Reverse the remaining 'n - d' elements
    _reverse_subarray(arr, d, n - 1)
    
    # 3. Reverse the entire array
    _reverse_subarray(arr, 0, n - 1)
    
    return arr

# Test cases
print("\nReversal Algorithm rotation:")
my_array = [1, 2, 3, 4, 5, 6, 7]
rotations = 3
print(f"Original: {my_array.copy()}, Rotations: {rotations}")
print(f"Rotated: {left_rotate_reversal(my_array.copy(), rotations)}")

my_array_2 = ['a', 'b', 'c', 'd', 'e', 'f']
rotations_2 = 4
print(f"Original: {my_array_2.copy()}, Rotations: {rotations_2}")
print(f"Rotated: {left_rotate_reversal(my_array_2.copy(), rotations_2)}")
```

```output
Reversal Algorithm rotation:
Original: [1, 2, 3, 4, 5, 6, 7], Rotations: 3
Rotated: [4, 5, 6, 7, 1, 2, 3]
Original: ['a', 'b', 'c', 'd', 'e', 'f'], Rotations: 4
Rotated: ['e', 'f', 'a', 'b', 'c', 'd']
```

## Right Rotation

Right rotation is simply the reverse of left rotation. A right rotation by `d` positions is equivalent to a left rotation by `n - d` positions. We can leverage the efficient algorithms developed for left rotation.

**How it works (using Left Rotation logic)**:
To perform a right rotation of `d` positions on an array of size `n`:
1.  Calculate the effective left rotation amount: `left_d = n - d`.
2.  Perform a left rotation using any of the O(n) algorithms (Juggling or Reversal) with `left_d`.

**Alternatively, using the Reversal Algorithm directly for right rotation**:
For a right rotation by `d` positions:
1.  Reverse the entire array `arr[0...n-1]`.
2.  Reverse the first `d` elements `arr[0...d-1]`.
3.  Reverse the remaining `n-d` elements `arr[d...n-1]`.

**Example walk-through (for `arr = [1, 2, 3, 4, 5, 6, 7]`, `d = 3` right rotation):**
Original: `[1, 2, 3, 4, 5, 6, 7]`

1.  Reverse entire array:
    `[7, 6, 5, 4, 3, 2, 1]`

2.  Reverse first `d=3` elements: `arr[0...2]`
    `[5, 6, 7 | 4, 3, 2, 1]`

3.  Reverse remaining `n-d=4` elements: `arr[3...6]`
    `[5, 6, 7 | 1, 2, 3, 4]`

This results in `[5, 6, 7, 1, 2, 3, 4]`, which is a 3-position right rotation.

**Time Complexity**: O(n)
**Space Complexity**: O(1)

**Example (Python):**

```python
# Reusing the _reverse_subarray helper from before

def right_rotate_reversal(arr, d):
    n = len(arr)
    if n == 0 or d == 0:
        return arr

    d = d % n # Normalize d
    
    # Handle edge case where d is 0 or n (no rotation needed)
    if d == 0:
        return arr

    # 1. Reverse the entire array
    _reverse_subarray(arr, 0, n - 1)
    
    # 2. Reverse the first 'd' elements (which are now the last 'd' of original)
    _reverse_subarray(arr, 0, d - 1)
    
    # 3. Reverse the remaining 'n - d' elements (which are now the first 'n-d' of original)
    _reverse_subarray(arr, d, n - 1)
    
    return arr

# Test cases
print("\nRight Rotation using Reversal Algorithm:")
my_array = [1, 2, 3, 4, 5, 6, 7]
rotations = 3 # Right rotate by 3
print(f"Original: {my_array.copy()}, Rotations: {rotations}")
print(f"Rotated: {right_rotate_reversal(my_array.copy(), rotations)}")

my_array_2 = [10, 20, 30, 40, 50]
rotations_2 = 2 # Right rotate by 2
print(f"Original: {my_array_2.copy()}, Rotations: {rotations_2}")
print(f"Rotated: {right_rotate_reversal(my_array_2.copy(), rotations_2)}")
```

```output
Right Rotation using Reversal Algorithm:
Original: [1, 2, 3, 4, 5, 6, 7], Rotations: 3
Rotated: [5, 6, 7, 1, 2, 3, 4]
Original: [10, 20, 30, 40, 50], Rotations: 2
Rotated: [40, 50, 10, 20, 30]
```

## Important Considerations & Edge Cases

*   **`d > n`**: Always normalize `d` by using `d = d % n`. If `d` becomes 0 after normalization, no rotation is needed.
*   **Empty Array**: If the array is empty (`n=0`), return it as is. Your code should handle `len(arr) == 0` gracefully.
*   **`d = 0` or `d = n`**: If `d` is 0 or a multiple of `n` (after normalization), the array remains unchanged. Your `d = d % n` step takes care of this.
*   **Single Element Array**: An array with a single element `[X]` will remain `[X]` regardless of `d`. The algorithms will naturally handle this.
*   **Modifying Original Array vs. New Array**: Be mindful if the problem requires modifying the array in-place or returning a new rotated array. Python's slicing `arr[d:] + arr[:d]` creates a *new* list, while the in-place Juggling and Reversal algorithms modify the original.

## Conclusion

Array rotations are more than just shifting elements; they're a great way to understand algorithm design trade-offs.

*   The **Naive (One-by-One)** approach is simple but inefficient (O(N*D)). Avoid it unless `D` is very small.
*   Using a **Temporary Array** is straightforward and improves efficiency to O(N) time, but uses O(D) or O(N) auxiliary space. This is often acceptable, especially with higher-level languages that manage memory well.
*   The **Juggling Algorithm** and **Reversal Algorithm** are the optimal solutions, both achieving O(N) time complexity and O(1) space complexity. They are crucial for interviews and scenarios where memory is constrained. The Juggling Algorithm is conceptually a bit harder to grasp, while the Reversal Algorithm is quite intuitive once you see the pattern.

For practical Python development, if in-place modification isn't strictly required, the slicing `arr[d:] + arr[:d]` is often the most concise and readable. However, understanding the in-place O(1) space algorithms is fundamental for any serious developer. Choose the right tool for the job based on performance requirements, memory constraints, and code readability.