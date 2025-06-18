---
title: Efficiently Printing Unique Rows in a Boolean Matrix
date: 2025-06-18T02:04:08.681Z
description: Dive into practical techniques for identifying and printing unique rows within a boolean (binary) matrix. We explore idiomatic Python, bit manipulation, and the powerful Trie data structure, complete with runnable code examples.
tags: [Python, Algorithms, Data Structures, Matrix, Boolean, Bit Manipulation, Deduplication]
categories: [Programming, Algorithms]
comments: true
---

Working with matrices is a fundamental part of many programming tasks, from image processing to data analysis. A specific variant, the *boolean matrix* (also known as a binary matrix), where every element is either `0` or `1`, appears frequently in scenarios like feature vectors, adjacency matrices, or representing sets.

A common challenge with any dataset is dealing with duplicates. For boolean matrices, this means identifying and printing only the rows that are unique. This isn't just an academic exercise; think about de-duplicating configurations, filtering unique patterns, or optimizing storage.

In this post, we'll explore several effective ways to tackle this problem, ranging from simple Pythonic approaches to more advanced data structures. We'll provide clear, runnable code examples for each.

Let's dive in!

## The Problem: Print Unique Rows

Given a boolean matrix, identify and print each row only once.

**Example Input Matrix:**

```
[
  [0, 1, 0, 0, 1],
  [1, 0, 1, 1, 0],
  [0, 1, 0, 0, 1],  <-- Duplicate of row 0
  [1, 0, 1, 0, 0],
  [0, 1, 0, 0, 1]   <-- Duplicate of row 0
]
```

**Expected Unique Rows Output:**

```
[
  [0, 1, 0, 0, 1],
  [1, 0, 1, 1, 0],
  [1, 0, 1, 0, 0]
]
```

## Approach 1: Hashing with a Set (Idiomatic Python)

This is often the most straightforward and Pythonic way to handle uniqueness. The core idea is to convert each row into a hashable type (like a `tuple` or a `string`) and then add it to a `set`. Sets, by definition, only store unique elements.

### Concept

1.  Initialize an empty `set` to store unique rows.
2.  Iterate through each row of the matrix.
3.  Convert the current row (which is a `list`) into a `tuple`. Lists are mutable and thus not hashable, but tuples are immutable and hashable.
4.  Add the tuple representation of the row to the `set`. If the row is already present, the set simply ignores the insertion.
5.  Finally, convert the unique rows back from tuples to lists (if desired) and print them.

### Example Code

```python
def print_unique_rows_set(matrix):
    """
    Prints unique rows from a boolean matrix using a Python set.
    Converts list rows to tuples for hashability.
    """
    if not matrix:
        print("Matrix is empty.")
        return

    unique_rows_set = set()
    print("--- Using Set (Idiomatic Python) ---")
    print("Processing matrix:")
    for row in matrix:
        print(f"  Row: {row}")
        # Convert list to tuple to make it hashable
        row_tuple = tuple(row)
        if row_tuple not in unique_rows_set:
            unique_rows_set.add(row_tuple)
            print(f"    -> Added unique row: {row_tuple}")
        else:
            print(f"    -> Row already seen (duplicate).")

    print("\nUnique Rows Found:")
    for u_row in sorted(list(unique_rows_set)): # Sorting for consistent output
        print(list(u_row))

# Sample boolean matrix
boolean_matrix_1 = [
    [0, 1, 0, 0, 1],
    [1, 0, 1, 1, 0],
    [0, 1, 0, 0, 1],
    [1, 0, 1, 0, 0],
    [0, 1, 0, 0, 1]
]

boolean_matrix_2 = [
    [1, 1, 1],
    [0, 0, 0],
    [1, 1, 1],
    [1, 0, 1],
    [0, 0, 0]
]

print_unique_rows_set(boolean_matrix_1)
print("\n" + "="*40 + "\n") # Separator
print_unique_rows_set(boolean_matrix_2)

```

### Sample Output

```output
--- Using Set (Idiomatic Python) ---
Processing matrix:
  Row: [0, 1, 0, 0, 1]
    -> Added unique row: (0, 1, 0, 0, 1)
  Row: [1, 0, 1, 1, 0]
    -> Added unique row: (1, 0, 1, 1, 0)
  Row: [0, 1, 0, 0, 1]
    -> Row already seen (duplicate).
  Row: [1, 0, 1, 0, 0]
    -> Added unique row: (1, 0, 1, 0, 0)
  Row: [0, 1, 0, 0, 1]
    -> Row already seen (duplicate).

Unique Rows Found:
[0, 1, 0, 0, 1]
[1, 0, 1, 0, 0]
[1, 0, 1, 1, 0]

========================================

--- Using Set (Idiomatic Python) ---
Processing matrix:
  Row: [1, 1, 1]
    -> Added unique row: (1, 1, 1)
  Row: [0, 0, 0]
    -> Added unique row: (0, 0, 0)
  Row: [1, 1, 1]
    -> Row already seen (duplicate).
  Row: [1, 0, 1]
    -> Added unique row: (1, 0, 1)
  Row: [0, 0, 0]
    -> Row already seen (duplicate).

Unique Rows Found:
[0, 0, 0]
[1, 0, 1]
[1, 1, 1]
```

### Analysis

*   **Pros**: Simple to implement, highly readable, and leverages Python's optimized built-in `set` operations. Average time complexity is O(M * N) where M is number of rows, N is number of columns, assuming good hash distribution.
*   **Cons**: Space complexity is O(M * N) in the worst case (all rows unique) to store the unique rows. Converting `list` to `tuple` for each row adds a small overhead.

## Approach 2: Bit Manipulation (Efficient for Narrow Matrices)

Since we're dealing with *boolean* matrices, each row is essentially a sequence of bits. If the number of columns (`N`) is relatively small (e.g., up to 64 for a 64-bit integer, or theoretically arbitrary in Python), we can represent each row as a single integer. This allows for extremely compact storage and very fast hashing.

### Concept

1.  Initialize an empty `set` to store unique *integer representations* of rows.
2.  Iterate through each row.
3.  For each row, convert the sequence of `0`s and `1`s into a single integer. For example, `[0, 1, 0, 0, 1]` can be `0 * 2^4 + 1 * 2^3 + 0 * 2^2 + 0 * 2^1 + 1 * 2^0 = 0 + 8 + 0 + 0 + 1 = 9`. This is essentially converting a binary string to its decimal equivalent.
4.  Add this integer to the `set`.
5.  If the integer is unique, store the original row (or reconstruct it later) in a list of unique rows.
6.  Finally, print the collected unique rows.

**Note**: This approach is most advantageous when `N` (number of columns) is small enough to fit within a native integer type (e.g., 32-bit or 64-bit). Python's integers handle arbitrary precision, so `N` isn't strictly limited by native word size, but the *idea* of packing bits is most powerful when it leverages fixed-size integer arithmetic.

### Example Code

```python
def row_to_int(row):
    """Converts a boolean row (list of 0s and 1s) to its integer representation."""
    int_val = 0
    for bit in row:
        int_val = (int_val << 1) | bit
    return int_val

def int_to_row(int_val, num_cols):
    """Converts an integer back to its boolean row representation."""
    row = [0] * num_cols
    for i in range(num_cols - 1, -1, -1):
        row[i] = int_val & 1
        int_val >>= 1
    return row

def print_unique_rows_bit_manipulation(matrix):
    """
    Prints unique rows from a boolean matrix using bit manipulation and a set of integers.
    """
    if not matrix:
        print("Matrix is empty.")
        return

    num_rows = len(matrix)
    num_cols = len(matrix[0]) if num_rows > 0 else 0

    if num_cols == 0:
        print("Matrix has rows but no columns.")
        return

    # Note: For very wide matrices (e.g., > 64 columns in C/C++/Java),
    # this bit manipulation approach becomes less practical as it exceeds
    # native integer sizes. Python's arbitrary precision integers mitigate this,
    # but the concept of compact bit packing is strongest for narrower matrices.

    unique_row_integers = set()
    unique_rows_list = [] # To preserve order of first appearance if needed, or just print from here

    print("--- Using Bit Manipulation ---")
    print(f"Processing matrix (Rows: {num_rows}, Cols: {num_cols}):")
    for r_idx, row in enumerate(matrix):
        print(f"  Row {r_idx}: {row}")
        row_as_int = row_to_int(row)
        print(f"    -> Integer representation: {row_as_int}")

        if row_as_int not in unique_row_integers:
            unique_row_integers.add(row_as_int)
            unique_rows_list.append(row) # Store original row or convert back from int_to_row(row_as_int, num_cols)
            print(f"    -> Added unique integer: {row_as_int}")
        else:
            print(f"    -> Row already seen (duplicate).")

    print("\nUnique Rows Found:")
    for u_row in unique_rows_list:
        print(u_row)

# Sample boolean matrix
boolean_matrix_1 = [
    [0, 1, 0, 0, 1], # 9
    [1, 0, 1, 1, 0], # 22
    [0, 1, 0, 0, 1], # 9
    [1, 0, 1, 0, 0], # 20
    [0, 1, 0, 0, 1]  # 9
]

boolean_matrix_2 = [
    [1, 1, 1], # 7
    [0, 0, 0], # 0
    [1, 1, 1], # 7
    [1, 0, 1], # 5
    [0, 0, 0]  # 0
]

print_unique_rows_bit_manipulation(boolean_matrix_1)
print("\n" + "="*40 + "\n") # Separator
print_unique_rows_bit_manipulation(boolean_matrix_2)

```

### Sample Output

```output
--- Using Bit Manipulation ---
Processing matrix (Rows: 5, Cols: 5):
  Row 0: [0, 1, 0, 0, 1]
    -> Integer representation: 9
    -> Added unique integer: 9
  Row 1: [1, 0, 1, 1, 0]
    -> Integer representation: 22
    -> Added unique integer: 22
  Row 2: [0, 1, 0, 0, 1]
    -> Integer representation: 9
    -> Row already seen (duplicate).
  Row 3: [1, 0, 1, 0, 0]
    -> Integer representation: 20
    -> Added unique integer: 20
  Row 4: [0, 1, 0, 0, 1]
    -> Integer representation: 9
    -> Row already seen (duplicate).

Unique Rows Found:
[0, 1, 0, 0, 1]
[1, 0, 1, 1, 0]
[1, 0, 1, 0, 0]

========================================

--- Using Bit Manipulation ---
Processing matrix (Rows: 5, Cols: 3):
  Row 0: [1, 1, 1]
    -> Integer representation: 7
    -> Added unique integer: 7
  Row 1: [0, 0, 0]
    -> Integer representation: 0
    -> Added unique integer: 0
  Row 2: [1, 1, 1]
    -> Integer representation: 7
    -> Row already seen (duplicate).
  Row 3: [1, 0, 1]
    -> Integer representation: 5
    -> Added unique integer: 5
  Row 4: [0, 0, 0]
    -> Integer representation: 0
    -> Row already seen (duplicate).

Unique Rows Found:
[1, 1, 1]
[0, 0, 0]
[1, 0, 1]
```

### Analysis

*   **Pros**: Very memory efficient for storing the unique row identifiers (just integers). Hashing integers is extremely fast. Time complexity remains O(M * N) for the conversion, but the set operations are very fast on integers.
*   **Cons**: The primary limitation is the number of columns, `N`. While Python's integers handle arbitrary size, conceptually, this approach truly shines when `N` fits within a standard machine word (e.g., 64 bits). For very wide matrices, the integer conversion itself can become costly, and `set` operations on truly massive integers might be slower than on simpler types. It's also less intuitive than the direct tuple conversion.

## Approach 3: Using a Trie (Prefix Tree)

A Trie (pronounced "try"), or prefix tree, is a tree-like data structure that stores a dynamic set of strings or sequences where the keys are usually characters. For our boolean matrix, each row can be seen as a sequence of `0`s and `1`s. Tries are excellent for checking the existence of sequences and can be particularly efficient if many rows share common prefixes.

### Concept

1.  **Trie Node**: Each node in the Trie will represent a bit (`0` or `1`). A node will have two children (one for `0`, one for `1`) and a flag `is_end_of_row` to indicate if a complete row ends at this node.
2.  **Insertion**: To insert a row, start at the Trie's root. For each bit in the row, traverse to the corresponding child. If a child doesn't exist, create it.
3.  **Uniqueness Check**: When you reach the end of a row (i.e., you've processed all its bits), check the `is_end_of_row` flag of the current node.
    *   If it's already `True`, the row is a duplicate.
    *   If it's `False`, set it to `True` and the row is unique.

### Example Code

```python
class TrieNode:
    def __init__(self):
        # Children for 0 and 1
        self.children = [None, None]
        # True if a row ends at this node
        self.is_end_of_row = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, row):
        """
        Inserts a boolean row into the Trie.
        Returns True if the row was new (unique), False if it was a duplicate.
        """
        current = self.root
        for bit in row:
            if current.children[bit] is None:
                current.children[bit] = TrieNode()
            current = current.children[bit]

        # After traversing the whole row, check if it's new
        if current.is_end_of_row:
            return False  # Row already exists
        else:
            current.is_end_of_row = True
            return True   # New unique row

def print_unique_rows_trie(matrix):
    """
    Prints unique rows from a boolean matrix using a Trie data structure.
    """
    if not matrix:
        print("Matrix is empty.")
        return

    trie = Trie()
    unique_rows_list = []

    print("--- Using Trie (Prefix Tree) ---")
    print("Processing matrix:")
    for row_idx, row in enumerate(matrix):
        print(f"  Row {row_idx}: {row}")
        if trie.insert(row):
            unique_rows_list.append(row)
            print(f"    -> Added unique row using Trie.")
        else:
            print(f"    -> Row already seen (duplicate).")

    print("\nUnique Rows Found:")
    for u_row in unique_rows_list:
        print(u_row)

# Sample boolean matrix
boolean_matrix_1 = [
    [0, 1, 0, 0, 1],
    [1, 0, 1, 1, 0],
    [0, 1, 0, 0, 1],
    [1, 0, 1, 0, 0],
    [0, 1, 0, 0, 1]
]

boolean_matrix_2 = [
    [1, 1, 1],
    [0, 0, 0],
    [1, 1, 1],
    [1, 0, 1],
    [0, 0, 0]
]

print_unique_rows_trie(boolean_matrix_1)
print("\n" + "="*40 + "\n") # Separator
print_unique_rows_trie(boolean_matrix_2)

```

### Sample Output

```output
--- Using Trie (Prefix Tree) ---
Processing matrix:
  Row 0: [0, 1, 0, 0, 1]
    -> Added unique row using Trie.
  Row 1: [1, 0, 1, 1, 0]
    -> Added unique row using Trie.
  Row 2: [0, 1, 0, 0, 1]
    -> Row already seen (duplicate).
  Row 3: [1, 0, 1, 0, 0]
    -> Added unique row using Trie.
  Row 4: [0, 1, 0, 0, 1]
    -> Row already seen (duplicate).

Unique Rows Found:
[0, 1, 0, 0, 1]
[1, 0, 1, 1, 0]
[1, 0, 1, 0, 0]

========================================

--- Using Trie (Prefix Tree) ---
Processing matrix:
  Row 0: [1, 1, 1]
    -> Added unique row using Trie.
  Row 1: [0, 0, 0]
    -> Added unique row using Trie.
  Row 2: [1, 1, 1]
    -> Row already seen (duplicate).
  Row 3: [1, 0, 1]
    -> Added unique row using Trie.
  Row 4: [0, 0, 0]
    -> Row already seen (duplicate).

Unique Rows Found:
[1, 1, 1]
[0, 0, 0]
[1, 0, 1]
```

### Analysis

*   **Pros**: No hashing involved, avoiding potential collisions (though rare with good hash functions). Efficient if many rows share prefixes, as common paths are reused. Time complexity for inserting M rows of length N is O(M * N) in the worst case (no shared prefixes), and potentially better if there are many shared prefixes.
*   **Cons**: More complex to implement than the hash-based approaches. Can consume more memory than simple hashing for storing many small nodes, especially if rows are very long and have few shared prefixes.

## Comparison and Trade-offs

| Feature / Approach | Hashing (Set of Tuples)       | Bit Manipulation (Set of Ints)   | Trie (Prefix Tree)              |
| :----------------- | :---------------------------- | :------------------------------- | :------------------------------ |
| **Simplicity**     | Very High (idiomatic Python)  | Moderate (requires conversion)   | Moderate to High (custom class) |
| **Performance (Time)** | O(M * N) average case           | O(M * N) (conversion + fast set) | O(M * N) worst case             |
| **Memory Usage**   | O(M * N) (storing unique tuples)| O(M) (storing unique integers)   | O(Total bits in unique rows)    |
| **Best For**       | General-purpose, clear code   | Narrow matrices, high performance| Matrices with many shared prefixes |
| **Scalability (N columns)** | Good, handles arbitrary width | Excellent for N <= 64, less so for very wide | Good, but memory can be an issue for very sparse/long unique paths |
| **Flexibility**    | Easily adaptable to non-boolean data | Specific to boolean/binary data  | Can be adapted to other sequence data (strings, numbers) |

## Conclusion

We've explored three robust methods for identifying and printing unique rows in a boolean matrix, each with its own strengths:

1.  **Hashing with a Set (of Tuples)**: This is generally the most straightforward and Pythonic approach. It's highly readable and efficient for most common use cases, making it an excellent default choice.
2.  **Bit Manipulation (with a Set of Integers)**: This method shines for "narrow" boolean matrices (e.g., up to 64 columns or so). It offers superior memory efficiency and very fast comparisons, leveraging the compact binary nature of the data.
3.  **Trie (Prefix Tree)**: While more complex to implement, the Trie is a powerful data structure, especially when rows might share many common prefixes. It can offer performance benefits in specific scenarios by reducing redundant comparisons.

As with most algorithmic problems, the "best" solution depends on your specific constraints: the size of your matrix, the expected distribution of rows (how many duplicates? how much prefix sharing?), and your priority (code simplicity, memory efficiency, raw speed).

For most everyday tasks in Python, the simple `set` approach with tuples will serve you well due to its clarity and reliability. For performance-critical applications with binary data, consider the bit manipulation approach. For highly specialized problems with significant prefix commonality, the Trie might be worth the implementation effort.

Choose wisely, and happy coding!