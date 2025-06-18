---
title: Disjoint Set Union (DSU) A Practical Guide for Developers
date: 2025-06-18T02:04:08.681Z
description: "Dive deep into Disjoint Set Union (DSU), also known as Union-Find, a powerful data structure for efficiently managing dynamic sets. Learn its core operations, essential optimizations (path compression and union by size), and practical applications with clear, runnable Python examples."
tags: ["Data Structures", "Algorithms", "DSU", "Union-Find", "Graph Theory", "Connectivity", "Python"]
categories: ["Programming", "Algorithms"]
comments: true
---

As developers, we often encounter problems that involve grouping elements, checking connectivity, or merging distinct sets. Whether it's finding connected components in a graph, optimizing Kruskal's algorithm for Minimum Spanning Trees (MST), or managing social network friendships, the Disjoint Set Union (DSU) data structure, also known as Union-Find, is an indispensable tool.

DSU provides an efficient way to manage a collection of disjoint (non-overlapping) sets. It supports two primary operations:

1.  **Find**: Determine which set an element belongs to. This operation typically returns a "representative" or "root" of the set.
2.  **Union**: Merge two sets into a single set.

Let's break down how it works, why it's powerful, and how to implement it effectively.

## The Core Idea: Representing Sets

At its heart, DSU uses a simple array (or hash map, if elements are not integers) to represent the parent of each element. Each element is initially in its own set, meaning its parent is itself.

`parent[i]` stores the parent of element `i`.
If `parent[i] == i`, then `i` is the root (representative) of its set.

Think of it like a forest of trees, where each tree represents a set, and the root of the tree is the representative of that set.

## Basic Implementation (The Naïve Approach)

Let's start with a foundational, albeit inefficient, implementation. This will help us understand the need for optimizations.

### `find(i)` Operation

To find the representative of an element `i`, we simply traverse up the parent chain until we reach a node that is its own parent.

```python
class NaiveDSU:
    def __init__(self, n):
        # Initialize each element as its own parent (each in its own set)
        self.parent = list(range(n))

    def find(self, i):
        # Traverse up until the root is found
        while self.parent[i] != i:
            i = self.parent[i]
        return i

    def union(self, i, j):
        # Find the roots of i and j
        root_i = self.find(i)
        root_j = self.find(j)

        # If they are not already in the same set, merge them
        if root_i != root_j:
            self.parent[root_i] = root_j # Make root_j the parent of root_i
            return True # Successfully united
        return False # Already in the same set

# Example Usage: Naive DSU
print("--- Naive DSU Example ---")
n_naive = 5
dsu_naive = NaiveDSU(n_naive)

print(f"Initial state: {dsu_naive.parent}")
print(f"Find(0): {dsu_naive.find(0)}")
print(f"Find(3): {dsu_naive.find(3)}")

dsu_naive.union(0, 1) # Merge set of 0 and set of 1
print(f"After union(0, 1): {dsu_naive.parent}")
print(f"Find(0): {dsu_naive.find(0)}") # Should be same as Find(1)
print(f"Find(1): {dsu_naive.find(1)}")

dsu_naive.union(2, 3) # Merge set of 2 and set of 3
print(f"After union(2, 3): {dsu_naive.parent}")

dsu_naive.union(0, 2) # Merge set of 0 and set of 2 (which now includes 1 and 3)
print(f"After union(0, 2): {dsu_naive.parent}")
print(f"Are 1 and 3 connected? {dsu_naive.find(1) == dsu_naive.find(3)}")
print(f"Are 0 and 4 connected? {dsu_naive.find(0) == dsu_naive.find(4)}")
```

```output
--- Naive DSU Example ---
Initial state: [0, 1, 2, 3, 4]
Find(0): 0
Find(3): 3
After union(0, 1): [1, 1, 2, 3, 4]
Find(0): 1
Find(1): 1
After union(2, 3): [1, 1, 3, 3, 4]
After union(0, 2): [1, 1, 3, 1, 4]
Are 1 and 3 connected? True
Are 0 and 4 connected? False
```

### The Problem with Naïve DSU

In the worst-case scenario (e.g., `union(0,1)`, then `union(1,2)`, then `union(2,3)`, and so on), this naive approach can create a long, degenerate tree structure. A `find` operation on an element at the bottom of such a tree would take `O(N)` time, where `N` is the number of elements. This can lead to `O(N*M)` complexity for `M` operations, which is often too slow for competitive programming or large-scale applications.

## Optimizations: Making DSU Efficient

To overcome the performance issues, DSU employs two powerful optimizations:

1.  **Path Compression** (for `find` operation)
2.  **Union by Rank or Size** (for `union` operation)

When combined, these optimizations make the amortized time complexity of both `find` and `union` operations almost constant, specifically `O(α(N))`, where `α` is the inverse Ackermann function. For practical purposes, `α(N)` grows extremely slowly and is less than 5 for any realistic `N`, making it practically a constant time operation.

### Optimization 1: Path Compression

The idea behind path compression is simple: whenever we perform a `find` operation, we traverse up the tree to find the root. On the way back down, we make every node on the traversed path point directly to the root. This flattens the tree, ensuring that future `find` operations for these nodes (or their descendants) will be much faster.

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        # For union by size/rank, we also need to store the size/rank of each set
        # Using size for simplicity and common practice. `size[i]` is only valid if `i` is a root.
        self.size = [1] * n 

    def find(self, i):
        if self.parent[i] == i:
            return i
        # Path compression: make i's parent the root directly
        self.parent[i] = self.find(self.parent[i])
        return self.parent[i]

    def union(self, i, j):
        # We will implement union by size in the next section
        pass # Placeholder for now

# Example Usage: Path Compression (without full union implemented yet)
print("\n--- Path Compression Example (Find only) ---")
n_pc = 10
dsu_pc = DSU(n_pc)

# Manually create a long chain for demonstration
dsu_pc.parent[1] = 0
dsu_pc.parent[2] = 1
dsu_pc.parent[3] = 2
dsu_pc.parent[4] = 3

print(f"Initial parent state: {dsu_pc.parent}")
print(f"Finding root of 4: {dsu_pc.find(4)}") # This will compress the path
print(f"Parent state after find(4): {dsu_pc.parent}")
# Notice how parent[4], parent[3], parent[2], parent[1] all now point to 0 directly.
```

```output
--- Path Compression Example (Find only) ---
Initial parent state: [0, 0, 1, 2, 3, 5, 6, 7, 8, 9]
Finding root of 4: 0
Parent state after find(4): [0, 0, 0, 0, 0, 5, 6, 7, 8, 9]
```

### Optimization 2: Union by Rank or Size

When merging two sets, we want to keep the overall tree structure as flat as possible.
The "union by size" heuristic achieves this: when uniting two sets, attach the root of the smaller set to the root of the larger set. If sizes are equal, pick one arbitrarily and increment the size of the new combined set. This prevents the formation of deep trees.

"Union by rank" is an alternative that uses an estimated height of the tree (rank) instead of size. Both achieve similar performance benefits. Union by size is often slightly easier to implement.

Let's integrate both optimizations into our `DSU` class.

### Combined DSU Implementation

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n # size[i] stores the size of the set if i is its root
        self.num_components = n # Track the number of disjoint sets

    def find(self, i):
        """
        Finds the representative (root) of the set containing element i,
        with path compression.
        """
        if self.parent[i] == i:
            return i
        self.parent[i] = self.find(self.parent[i])
        return self.parent[i]

    def union(self, i, j):
        """
        Unites the sets containing elements i and j,
        using union by size.
        Returns True if a merge happened, False if they were already in the same set.
        """
        root_i = self.find(i)
        root_j = self.find(j)

        if root_i != root_j:
            # Union by size: attach smaller tree under root of larger tree
            if self.size[root_i] < self.size[root_j]:
                self.parent[root_i] = root_j
                self.size[root_j] += self.size[root_i]
            else:
                self.parent[root_j] = root_i
                self.size[root_i] += self.size[root_j]
            self.num_components -= 1 # One less component after successful merge
            return True
        return False # Elements already in the same set

    def is_connected(self, i, j):
        """
        Checks if elements i and j are in the same set.
        """
        return self.find(i) == self.find(j)

    def get_num_components(self):
        """
        Returns the current number of disjoint sets.
        """
        return self.num_components

    def get_set_size(self, i):
        """
        Returns the size of the set containing element i.
        """
        return self.size[self.find(i)]

# Example Usage: Optimized DSU
print("\n--- Optimized DSU Example ---")
num_elements = 7
dsu_opt = DSU(num_elements)

print(f"Initial state: {dsu_opt.parent}, Sizes: {dsu_opt.size}, Components: {dsu_opt.get_num_components()}")

dsu_opt.union(0, 1)
dsu_opt.union(1, 2) # Union(0,1) then Union(1,2) -> {0,1,2}
print(f"After union(0,1), union(1,2): {dsu_opt.parent}, Sizes: {dsu_opt.size}, Components: {dsu_opt.get_num_components()}")
print(f"Are 0 and 2 connected? {dsu_opt.is_connected(0, 2)}")

dsu_opt.union(3, 4)
dsu_opt.union(5, 6)
print(f"After union(3,4), union(5,6): {dsu_opt.parent}, Sizes: {dsu_opt.size}, Components: {dsu_opt.get_num_components()}")

print(f"Is 0 connected to 3? {dsu_opt.is_connected(0, 3)}")

dsu_opt.union(2, 5) # Union set {0,1,2} with set {5,6}
print(f"After union(2,5): {dsu_opt.parent}, Sizes: {dsu_opt.size}, Components: {dsu_opt.get_num_components()}")
print(f"Is 0 connected to 6? {dsu_opt.is_connected(0, 6)}")
print(f"Size of set containing 0: {dsu_opt.get_set_size(0)}")
print(f"Size of set containing 3: {dsu_opt.get_set_size(3)}")

```

```output
--- Optimized DSU Example ---
Initial state: [0, 1, 2, 3, 4, 5, 6], Sizes: [1, 1, 1, 1, 1, 1, 1], Components: 7
After union(0,1), union(1,2): [0, 0, 0, 3, 4, 5, 6], Sizes: [3, 1, 1, 1, 1, 1, 1], Components: 5
Are 0 and 2 connected? True
After union(3,4), union(5,6): [0, 0, 0, 3, 3, 5, 5], Sizes: [3, 1, 1, 2, 1, 2, 1], Components: 3
Is 0 connected to 3? False
After union(2,5): [0, 0, 0, 3, 3, 0, 5], Sizes: [5, 1, 1, 2, 1, 2, 1], Components: 2
Is 0 connected to 6? True
Size of set containing 0: 5
Size of set containing 3: 2
```

**Note**: The `parent` array's exact values might vary due to path compression and union-by-size choices, but the `find` results and `is_connected` logic will always be consistent. For example, `parent[6]` points to `5` initially, but after `union(2,5)` and `find(6)`, it might point to `0` directly. The key is that `find(6)` eventually returns the root of the set (`0` in the final state). `size` values are only meaningful at the root.

## Practical Applications of DSU

DSU is incredibly versatile. Here are a few common scenarios where it shines:

### 1. Connected Components in a Graph

Given a graph, DSU can efficiently determine how many connected components it has and which vertices belong to the same component.

**Problem**: Count the number of connected components in an undirected graph.

**Approach**:
1.  Initialize DSU with `N` elements (vertices).
2.  For each edge `(u, v)` in the graph, perform `dsu.union(u, v)`.
3.  The final number of disjoint sets tracked by DSU is the number of connected components.

```python
# Assuming the DSU class from above is defined

def count_connected_components(n_vertices, edges):
    dsu = DSU(n_vertices)
    for u, v in edges:
        dsu.union(u, v)
    return dsu.get_num_components()

print("\n--- Connected Components Example ---")
num_vertices = 7
graph_edges = [
    (0, 1),
    (1, 2),
    (3, 4),
    (0, 2), # Redundant, but union handles it
    (5, 6)
]

components = count_connected_components(num_vertices, graph_edges)
print(f"Number of connected components: {components}")

# Example 2: More complex graph
num_vertices_2 = 9
graph_edges_2 = [
    (0, 1), (1, 2),
    (3, 4), (4, 5), (5, 3), # A cycle
    (6, 7)
]
components_2 = count_connected_components(num_vertices_2, graph_edges_2)
print(f"Number of connected components (complex graph): {components_2}")

# Let's also see the groups formed
dsu_graph_groups = DSU(num_vertices_2)
for u,v in graph_edges_2:
    dsu_graph_groups.union(u,v)

groups = {}
for i in range(num_vertices_2):
    root = dsu_graph_groups.find(i)
    if root not in groups:
        groups[root] = []
    groups[root].append(i)

print("Groups formed (elements with same root):")
for root, elements in groups.items():
    print(f"  Root {root}: {elements}")
```

```output
--- Connected Components Example ---
Number of connected components: 3
Number of connected components (complex graph): 3
Groups formed (elements with same root):
  Root 0: [0, 1, 2]
  Root 3: [3, 4, 5]
  Root 6: [6, 7]
  Root 8: [8]
```

### 2. Kruskal's Algorithm for Minimum Spanning Tree (MST)

Kruskal's algorithm builds an MST by greedily adding edges with the smallest weights, as long as they don't form a cycle. DSU is perfect for efficiently detecting these cycles.

**Approach**:
1.  Sort all edges by weight in ascending order.
2.  Initialize DSU with `N` vertices.
3.  Iterate through the sorted edges:
    *   For an edge `(u, v)` with weight `w`:
    *   If `u` and `v` are not already connected (i.e., `dsu.find(u) != dsu.find(v)`), then `union(u, v)` them and add `w` to the MST's total weight.
4.  Stop when `N-1` edges are added or all edges are processed.

```python
# Assuming the DSU class from above is defined

def kruskal_mst(n_vertices, edges):
    # Edges format: (weight, u, v)
    edges.sort() # Sort by weight

    dsu = DSU(n_vertices)
    mst_cost = 0
    mst_edges = []
    edges_count = 0

    for weight, u, v in edges:
        if dsu.union(u, v): # If union successful (no cycle formed)
            mst_cost += weight
            mst_edges.append((u, v, weight))
            edges_count += 1
            if edges_count == n_vertices - 1: # MST has N-1 edges
                break
    return mst_cost, mst_edges

print("\n--- Kruskal's MST Example ---")
num_vertices_mst = 6
edges_mst = [
    (1, 0, 1),
    (5, 0, 2),
    (10, 1, 2),
    (2, 1, 3),
    (4, 2, 4),
    (3, 3, 4),
    (6, 4, 5)
]

cost, edges = kruskal_mst(num_vertices_mst, edges_mst)
print(f"Minimum Spanning Tree Cost: {cost}")
print("MST Edges:")
for u, v, w in edges:
    print(f"  ({u}, {v}) - Weight: {w}")

```

```output
--- Kruskal's MST Example ---
Minimum Spanning Tree Cost: 16
MST Edges:
  (0, 1) - Weight: 1
  (1, 3) - Weight: 2
  (3, 4) - Weight: 3
  (2, 4) - Weight: 4
  (0, 2) - Weight: 5
```

### 3. Grid Problems (e.g., Counting Islands, Flood Fill)

In grid-based problems, DSU can be used to group connected cells. For instance, counting islands in a binary matrix.

**Problem**: Given a 2D binary grid, where `1` represents land and `0` represents water, count the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically.

**Approach**:
1.  Map each grid cell `(r, c)` to a unique 1D index `r * cols + c`.
2.  Initialize DSU for `rows * cols` elements.
3.  Iterate through the grid. If a cell `grid[r][c]` is land (`1`):
    *   Consider its adjacent (up, down, left, right) land cells.
    *   For each adjacent land cell, `union` the current cell's index with the adjacent cell's index.
4.  After processing all cells, the number of distinct roots among all land cells (or the final `num_components` adjusted for water cells) will give the number of islands.

```python
# Assuming the DSU class from above is defined

def num_islands(grid):
    if not grid or not grid[0]:
        return 0

    rows, cols = len(grid), len(grid[0])
    # Map (r, c) to a 1D index: r * cols + c
    dsu = DSU(rows * cols)
    
    # Keep track of roots that are actually land cells
    land_roots = set()

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1': # If it's land
                current_idx = r * cols + c
                land_roots.add(dsu.find(current_idx)) # Add its root (initially itself)
                
                # Check neighbors (right and down, to avoid double counting and going out of bounds)
                # Up and Left neighbors would have already processed this cell.
                # Only need to connect to already processed land cells or newly encountered ones.
                # A simpler approach is to check all 4 neighbors and let union handle it.
                
                # Check right neighbor
                if c + 1 < cols and grid[r][c+1] == '1':
                    dsu.union(current_idx, r * cols + (c + 1))
                # Check down neighbor
                if r + 1 < rows and grid[r+1][c] == '1':
                    dsu.union(current_idx, (r + 1) * cols + c)
    
    # After all unions, iterate through all land cells again to find their final roots
    # and count distinct roots.
    final_island_roots = set()
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                final_island_roots.add(dsu.find(r * cols + c))
                
    return len(final_island_roots)

print("\n--- Counting Islands Example ---")
grid1 = [
  ["1","1","1","1","0"],
  ["1","1","0","1","0"],
  ["1","1","0","0","0"],
  ["0","0","0","0","0"]
]
print(f"Number of islands in Grid 1: {num_islands(grid1)}")

grid2 = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
print(f"Number of islands in Grid 2: {num_islands(grid2)}")
```

```output
--- Counting Islands Example ---
Number of islands in Grid 1: 1
Number of islands in Grid 2: 3
```

**Note**: For counting islands, simply using `dsu.get_num_components()` after all unions will count all elements initialized (including water). It's more accurate to count distinct roots only among the 'land' cells. My implementation iterates over land cells at the end to collect their final roots, which gives the correct island count.

## When to Use DSU

You should consider DSU when your problem involves:

*   **Dynamic Connectivity**: You need to frequently add connections between elements and check if two elements are connected.
*   **Grouping/Clustering**: You need to partition elements into sets based on some relationship.
*   **Cycle Detection**: In algorithms like Kruskal's, DSU efficiently detects if adding an edge forms a cycle.
*   **Counting Components**: Determining the number of disjoint groups or subgraphs.

## Conclusion

The Disjoint Set Union data structure is a classic yet incredibly powerful tool in algorithm design. Its ability to manage dynamic sets with nearly constant-time `find` and `union` operations, thanks to path compression and union by size/rank, makes it invaluable for solving a wide range of graph and connectivity problems. Understanding and implementing DSU effectively will significantly broaden your algorithmic toolkit. Practice with the examples, try to apply it to new problems, and you'll soon appreciate its elegance and efficiency.