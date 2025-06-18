---
title: Introduction To Heaps
date: 2025-06-18T02:04:08.681Z
description: A detailed introduction to heaps, covering their fundamental properties, types, common representations, core operations, and practical applications in data structures and algorithms. Learn by example with Python code.
tags: ["data structures", "algorithms", "heaps", "priority queue", "computer science"]
categories: ["Data Structures & Algorithms"]
comments: true
---

Heaps are a fundamental data structure in computer science, often misunderstood or conflated with general trees. While they are a type of tree, their specific properties make them exceptionally powerful for certain tasks, especially when you need quick access to the minimum or maximum element.

Let's demystify heaps.

## What is a Heap?

At its core, a **heap** is a specialized tree-based data structure that satisfies the "heap property." Unlike general binary search trees (BSTs) which maintain a specific order across the entire tree (left child < parent < right child), a heap only ensures a specific relationship between a parent and its direct children.

The most common implementation of a heap is a **binary heap**, which is what we'll focus on.

### Why Do We Need Heaps?

Imagine you have a stream of tasks, each with a priority, and you always need to process the highest (or lowest) priority task next.
*   A sorted list would require O(N) to insert and O(1) to extract, or O(log N) for insertion if kept sorted in a specific structure, but then O(N) shifts.
*   A linked list offers O(1) insert but O(N) to find the min/max.
*   A BST offers O(log N) for insertion and deletion on average, but finding the min/max still requires traversing the leftmost/rightmost path.

Heaps provide a sweet spot: **O(1) access to the min/max element** and **O(log N) for insertion and deletion** of elements. This efficiency is precisely what makes them invaluable for priority queues, certain sorting algorithms, and graph algorithms.

## Heap Properties

A binary heap must satisfy two critical properties:

### 1. Shape Property: A Complete Binary Tree

A binary heap is always a **complete binary tree**. This means:
*   Every level, except possibly the last, is completely filled.
*   All nodes at the last level are as far left as possible.

This property ensures that the tree is relatively "bushy" and compact, which is crucial for its efficient array representation.

```
       A
      / \
     B   C
    / \ / \
   D  E F  G

This is a complete binary tree. All levels are full.

       A
      / \
     B   C
    / \
   D  E

This is also a complete binary tree. The last level (D, E) is filled from left to right.

       A
      / \
     B   C
      \
       E

This is NOT a complete binary tree. Node E is not as far left as possible (B only has a right child, but no left child).
```

### 2. Heap Property: Parent-Child Relationship

This property defines the order within the heap. There are two types:

#### Min-Heap
In a **Min-Heap**, for every node `N` and its child `C`, the value of `N` is less than or equal to the value of `C`. This implies the smallest element is always at the root.

```
      10
     /  \
    20   30
   / \  / \
  40 50 60 70
```

#### Max-Heap
In a **Max-Heap**, for every node `N` and its child `C`, the value of `N` is greater than or equal to the value of `C`. This implies the largest element is always at the root.

```
      70
     /  \
    60   50
   / \  / \
  40 30 20 10
```

**Note:** Unlike BSTs where *all* elements in the left subtree are smaller and *all* elements in the right subtree are larger, a heap only guarantees the relationship between a parent and its direct children. There's no specific order between siblings or elements in different subtrees beyond the parent-child constraint.

## Heap Representation: The Array Trick

One of the most elegant aspects of a binary heap is that it can be efficiently stored in a simple array. Because it's a complete binary tree, we can map nodes to array indices without wasting space.

If a node is at index `i` (0-indexed):
*   Its **left child** is at index `2 * i + 1`
*   Its **right child** is at index `2 * i + 2`
*   Its **parent** is at index `(i - 1) // 2` (integer division)

Let's see an example:

```
Consider this Min-Heap:

          10 (index 0)
         /  \
(index 1) 20   30 (index 2)
         / \  / \
(idx 3) 40 50 60 70 (idx 6)

Represented as an array: `[10, 20, 30, 40, 50, 60, 70]`
```

Let's verify the formulas for node `20` (at index `1`):
*   Parent of `20` (index `1`): `(1 - 1) // 2 = 0 // 2 = 0`. Correct, its parent is `10` at index `0`.
*   Left child of `20` (index `1`): `2 * 1 + 1 = 3`. Correct, `40` at index `3`.
*   Right child of `20` (index `1`): `2 * 1 + 2 = 4`. Correct, `50` at index `4`.

This array representation is what makes heap operations so efficient, as we don't need to deal with explicit pointers.

## Basic Heap Operations

Most programming languages provide built-in heap (priority queue) implementations or modules. In Python, the `heapq` module provides a min-heap implementation.

### 1. Insertion (`heappush`)

To insert a new element:
1.  Add the new element to the end of the array (maintaining the complete binary tree property).
2.  **"Bubble Up" (heapify-up)**: Compare the new element with its parent. If it violates the heap property (e.g., in a min-heap, child is smaller than parent), swap them. Continue this process upwards until the heap property is restored or the root is reached.

**Complexity**: O(log N), because in the worst case, the new element might bubble up from the last level to the root, which is the height of the tree (log N).

```python
import heapq

# Initialize an empty min-heap
min_heap = []
print(f"Initial heap: {min_heap}")

# Insert elements
print("\n--- Inserting elements ---")
elements_to_insert = [5, 1, 8, 3, 2, 7]

for elem in elements_to_insert:
    print(f"Inserting: {elem}")
    heapq.heappush(min_heap, elem)
    print(f"Heap after push: {min_heap}")

print(f"\nFinal heap after insertions: {min_heap}")
# Note: The array representation of the heap might not look sorted,
# but the heap property (parent <= children) is maintained.
# The smallest element is always at index 0.
```
```output
Initial heap: []

--- Inserting elements ---
Inserting: 5
Heap after push: [5]
Inserting: 1
Heap after push: [1, 5]
Inserting: 8
Heap after push: [1, 5, 8]
Inserting: 3
Heap after push: [1, 3, 8, 5]
Inserting: 2
Heap after push: [1, 2, 8, 5, 3]
Inserting: 7
Heap after push: [1, 2, 7, 5, 3, 8]

Final heap after insertions: [1, 2, 7, 5, 3, 8]
```

### 2. Extraction (`heappop`)

To extract the minimum (from a min-heap) or maximum (from a max-heap) element:
1.  The element to be removed is always at the root (index 0).
2.  Replace the root with the last element in the heap array.
3.  Remove the last element (which was moved).
4.  **"Bubble Down" (heapify-down)**: Compare the new root with its children. If it violates the heap property (e.g., in a min-heap, root is larger than one of its children), swap it with the *smaller* child (for a min-heap) or *larger* child (for a max-heap). Continue this process downwards until the heap property is restored or a leaf node is reached.

**Complexity**: O(log N), for the same reason as insertion.

```python
import heapq

# Re-using the heap from the previous example: [1, 2, 7, 5, 3, 8]
min_heap = [1, 2, 7, 5, 3, 8]
print(f"Heap before pops: {min_heap}")

# Extract elements
print("\n--- Extracting elements ---")
while min_heap:
    smallest_item = heapq.heappop(min_heap)
    print(f"Popped: {smallest_item}")
    print(f"Heap after pop: {min_heap}")

print(f"\nFinal heap after all pops: {min_heap}")
```
```output
Heap before pops: [1, 2, 7, 5, 3, 8]

--- Extracting elements ---
Popped: 1
Heap after pop: [2, 3, 7, 5, 8]
Popped: 2
Heap after pop: [3, 5, 7, 8]
Popped: 3
Heap after pop: [5, 8, 7]
Popped: 5
Heap after pop: [7, 8]
Popped: 7
Heap after pop: [8]
Popped: 8
Heap after pop: []

Final heap after all pops: []
```

### 3. Peeking

To peek at the minimum (or maximum) element without removing it, simply access the element at index 0 of the heap array.

**Complexity**: O(1).

```python
import heapq

min_heap = [1, 2, 7, 5, 3, 8]
print(f"Current heap: {min_heap}")

if min_heap:
    smallest_item = min_heap[0] # Directly access the root
    print(f"Smallest item (peeked): {smallest_item}")
    print(f"Heap after peek: {min_heap}")
else:
    print("Heap is empty.")
```
```output
Current heap: [1, 2, 7, 5, 3, 8]
Smallest item (peeked): 1
Heap after peek: [1, 2, 7, 5, 3, 8]
```

### 4. Heapify (Building a Heap)

To convert an arbitrary list into a heap in-place, you can use the `heapify` operation. This process generally involves "bubbling down" elements from the last non-leaf node up to the root.

**Complexity**: O(N). While it might seem like it should be O(N log N) (N insertions, each log N), the total number of swaps due to the structure averages out to O(N).

```python
import heapq

# An unsorted list
data = [9, 2, 7, 1, 5, 3, 8, 4, 6]
print(f"Original list: {data}")

# Convert the list into a heap in-place
heapq.heapify(data)
print(f"Heapified list: {data}")

# Verify by popping elements (they should come out in sorted order)
print("\n--- Popping elements from the heapified list ---")
while data:
    print(f"Popped: {heapq.heappop(data)}")
```
```output
Original list: [9, 2, 7, 1, 5, 3, 8, 4, 6]
Heapified list: [1, 2, 3, 4, 5, 7, 8, 9, 6]

--- Popping elements from the heapified list ---
Popped: 1
Popped: 2
Popped: 3
Popped: 4
Popped: 5
Popped: 6
Popped: 7
Popped: 8
Popped: 9
```

## Common Use Cases

Heaps are incredibly versatile and form the backbone of many efficient algorithms.

### 1. Priority Queues

This is the most common and intuitive application. A priority queue is an abstract data type that provides efficient access to the item with the highest (or lowest) priority. Heaps are the perfect underlying data structure for this.

**Example**: Task scheduling.

```python
import heapq

# Simulate tasks with (priority, task_name) tuples
# Lower number means higher priority (min-heap will prioritize lower numbers)
tasks = []

print("--- Scheduling Tasks ---")
heapq.heappush(tasks, (3, "Write blog post"))
print(f"Added: Write blog post. Current queue: {tasks}")

heapq.heappush(tasks, (1, "Fix critical bug"))
print(f"Added: Fix critical bug. Current queue: {tasks}")

heapq.heappush(tasks, (2, "Review PR"))
print(f"Added: Review PR. Current queue: {tasks}")

heapq.heappush(tasks, (1, "Deploy hotfix")) # Same priority as 'Fix critical bug'
print(f"Added: Deploy hotfix. Current queue: {tasks}")

print("\n--- Processing Tasks ---")
while tasks:
    priority, task_name = heapq.heappop(tasks)
    print(f"Processing task: '{task_name}' with priority {priority}")
    print(f"Remaining queue: {tasks}")

print("\nAll tasks processed!")
```
```output
--- Scheduling Tasks ---
Added: Write blog post. Current queue: [(3, 'Write blog post')]
Added: Fix critical bug. Current queue: [(1, 'Fix critical bug'), (3, 'Write blog post')]
Added: Review PR. Current queue: [(1, 'Fix critical bug'), (3, 'Write blog post'), (2, 'Review PR')]
Added: Deploy hotfix. Current queue: [(1, 'Deploy hotfix'), (1, 'Fix critical bug'), (2, 'Review PR'), (3, 'Write blog post')]

--- Processing Tasks ---
Processing task: 'Deploy hotfix' with priority 1
Remaining queue: [(1, 'Fix critical bug'), (3, 'Write blog post'), (2, 'Review PR')]
Processing task: 'Fix critical bug' with priority 1
Remaining queue: [(2, 'Review PR'), (3, 'Write blog post')]
Processing task: 'Review PR' with priority 2
Remaining queue: [(3, 'Write blog post')]
Processing task: 'Write blog post' with priority 3
Remaining queue: []

All tasks processed!
```
**Note:** When priorities are equal, `heapq` processes elements based on their insertion order (or the comparison of the secondary elements in the tuple).

### 2. Heapsort

Heapsort is an efficient, in-place comparison sorting algorithm. It works by first building a max-heap from the input data, then repeatedly extracting the maximum element (which is at the root) and placing it at the end of the array, then re-heapifying the remaining elements.

*   Building the heap: O(N)
*   N extractions: N * O(log N) = O(N log N)
*   **Total Complexity**: O(N log N)

While Python's `heapq` module doesn't directly provide a heapsort function, you can implement it using `heapify` and repeated `heappop` operations, as demonstrated in the "Heapify" example which effectively sorts the data.

### 3. Finding the k-th Smallest/Largest Element

Heaps provide an efficient way to find the k-th smallest or largest element in a collection, or the `k` smallest/largest elements.

**Scenario**: Find the 3 largest numbers in a stream of numbers.
You can use a min-heap of size `k`.
*   Iterate through the numbers.
*   If the heap size is less than `k`, push the number.
*   If the heap size is `k` and the current number is greater than the smallest element in the heap (the root), pop the smallest and push the current number.

```python
import heapq

def find_k_largest(numbers, k):
    """Finds the k largest numbers using a min-heap."""
    if k <= 0:
        return []

    min_heap_of_k_largest = []

    for num in numbers:
        if len(min_heap_of_k_largest) < k:
            heapq.heappush(min_heap_of_k_largest, num)
        elif num > min_heap_of_k_largest[0]: # If current num is greater than the smallest in our heap
            heapq.heappushpop(min_heap_of_k_largest, num) # Pop smallest, then push new num

    # The heap now contains the k largest numbers.
    # To get them in descending order, we can pop them all.
    # Note: they are not necessarily sorted *within* the heap array representation.
    return sorted(min_heap_of_k_largest, reverse=True)

data_stream = [3, 2, 1, 5, 6, 4, 10, 7, 9, 8]
k_value = 3

result = find_k_largest(data_stream, k_value)
print(f"Original data stream: {data_stream}")
print(f"The {k_value} largest numbers are: {result}")

k_value = 5
result = find_k_largest(data_stream, k_value)
print(f"The {k_value} largest numbers are: {result}")

empty_stream = []
k_value = 2
result = find_k_largest(empty_stream, k_value)
print(f"For empty stream, k={k_value}, result: {result}")
```
```output
Original data stream: [3, 2, 1, 5, 6, 4, 10, 7, 9, 8]
The 3 largest numbers are: [10, 9, 8]
The 5 largest numbers are: [10, 9, 8, 7, 6]
For empty stream, k=2, result: []
```

### 4. Graph Algorithms

Heaps are integral to efficient implementations of several graph algorithms:
*   **Dijkstra's Algorithm**: For finding the shortest path from a single source to all other vertices in a graph with non-negative edge weights. A priority queue (implemented with a min-heap) is used to efficiently retrieve the unvisited vertex with the smallest tentative distance.
*   **Prim's Algorithm**: For finding a minimum spanning tree in an undirected, weighted graph. Similar to Dijkstra's, a priority queue is used to select the next edge to add.

## When to Use Heaps (and When Not To)

### Use Heaps When:

*   You frequently need to find the minimum or maximum element in a collection.
*   You need to add or remove elements efficiently while maintaining the min/max property.
*   You are implementing a priority queue.
*   You are dealing with streaming data and need to keep track of the `k` smallest/largest elements.
*   The data structure doesn't need to be fully sorted, just its extremes efficiently accessible.

### Avoid Heaps When:

*   You need to quickly search for an arbitrary element (e.g., "Is `X` in the heap?"). Heaps don't support efficient lookup of non-root elements (O(N) in worst case). A hash table or a balanced BST would be better.
*   You need to iterate through elements in a fully sorted order (unless you're popping all elements, which effectively sorts them, but a dedicated sorting algorithm might be clearer).
*   You need to find the `i`-th smallest/largest element quickly for arbitrary `i` (without repeatedly popping). For example, finding the median efficiently in a stream often involves two heaps.

## Conclusion

Heaps are a fascinating and highly practical data structure. Understanding their core properties—the complete binary tree shape and the heap property—is key to grasping why they are so efficient. By leveraging the simple array representation, we can build robust priority queues and solve a variety of algorithmic problems with optimal time complexity.

Next time you need to prioritize tasks, find extremes, or optimize a graph algorithm, think heaps! They're a fundamental tool in any developer's arsenal.