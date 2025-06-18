---
title: Demystifying Flood Fill - A Practical Guide for Developers
date: 2025-06-18T02:04:08.681Z
description: Dive deep into the Flood Fill algorithm, understanding its core mechanics, recursive and iterative implementations, and practical applications in image processing and game development. Learn with hands-on Python examples.
tags: ["algorithm", "flood-fill", "recursion", "bfs", "dfs", "image-processing", "graph-theory", "python"]
categories: ["Algorithms", "Programming"]
comments: true
---

The Flood Fill algorithm is one of those classic computer science concepts that feels almost magical when you first encounter it. It's the engine behind the "paint bucket" tool in image editors, a core mechanic in many puzzle games, and a fundamental building block for various graph traversal problems.

Despite its powerful applications, Flood Fill is surprisingly simple at its core. If you understand basic grid traversal (like navigating a maze) and recursion or queues, you're already halfway there. This post will break down Flood Fill, explain its common implementations, and provide practical, runnable Python examples for each.

Let's dive in!

## What is Flood Fill? The Core Idea

Imagine you have a grid, like a checkerboard, where each square has a certain "color" or "value." You pick a starting square and want to change its color, and also change the color of all *connected* squares that have the *same original color* as your starting square. This "spread" of color is what Flood Fill does.

Think of it like pouring water into a connected region. The water spreads to fill all adjacent spaces that aren't blocked by a different type of boundary.

### Key Concepts:
*   **Target Color (Old Color):** The color of the starting pixel and all connected pixels that you want to replace.
*   **Replacement Color (New Color):** The color you want to fill the region with.
*   **Starting Point (Seed):** The initial pixel from which the fill operation begins.
*   **Connectivity:** How "adjacent" is defined. Typically, this is 4-way (up, down, left, right) or 8-way (including diagonals).

The algorithm proceeds by:
1.  Checking the starting point. If it's already the `replacement color` or not the `target color`, do nothing.
2.  Change the color of the current pixel to the `replacement color`.
3.  Look at its neighbors. For each neighbor that is within bounds and has the `target color`, apply the same process (step 2 and 3).

This recursive nature is what makes it so elegant.

## Implementation Strategies

There are two primary ways to implement Flood Fill: recursively (Depth-First Search style) and iteratively (Breadth-First Search style). Both achieve the same result but have different performance characteristics and considerations.

For our examples, we'll represent the grid as a list of lists of characters (e.g., `'W'` for white, `'B'` for black, etc.).

Let's define a helper function to visualize our grid:

```python
def print_grid(grid):
    """Prints the 2D grid."""
    for row in grid:
        print(" ".join(row))
    print("-" * 15) # Separator for clarity
```

### 1. Recursive Flood Fill (DFS Approach)

The recursive approach is often the most intuitive and concise. It mirrors the definition of Flood Fill directly: "fill this cell, then recursively fill its valid neighbors." This is essentially a Depth-First Search (DFS) traversal.

**How it works:**
1.  A function takes the grid, current row `r`, current column `c`, `target_color`, and `new_color`.
2.  It first checks base cases:
    *   If `(r, c)` is out of bounds, stop.
    *   If the cell at `(r, c)` is *not* the `target_color`, stop (it's a boundary or already filled).
    *   If the cell at `(r, c)` is *already* the `new_color`, stop (to prevent infinite loops in some edge cases or redundant processing).
3.  If none of the above, change `grid[r][c]` to `new_color`.
4.  Recursively call the function for all valid (4-way) neighbors: `(r+1, c)`, `(r-1, c)`, `(r, c+1)`, `(r, c-1)`.

**Pros:**
*   Elegant and easy to read.
*   Minimal explicit data structures needed (the call stack handles state).

**Cons:**
*   **Stack Overflow Risk:** For very large connected regions, the recursion depth can exceed Python's default recursion limit (or the system's stack size), leading to a `RecursionError`.

#### Code Example: Recursive Flood Fill (4-way)

```python
# Initial grid for DFS example
grid_dfs = [
    ['W', 'W', 'W', 'W', 'W', 'W', 'W'],
    ['W', 'B', 'B', 'W', 'B', 'B', 'W'],
    ['W', 'B', 'W', 'W', 'B', 'W', 'W'],
    ['W', 'B', 'B', 'W', 'B', 'B', 'W'],
    ['W', 'W', 'W', 'W', 'W', 'W', 'W']
]

rows_dfs = len(grid_dfs)
cols_dfs = len(grid_dfs[0])

def flood_fill_recursive(grid, r, c, target_color, new_color):
    """
    Performs a 4-way flood fill recursively.
    grid: The 2D list representing the image/grid.
    r, c: Current row and column.
    target_color: The color to replace.
    new_color: The color to fill with.
    """
    # Base cases for recursion termination
    # 1. Out of bounds
    if r < 0 or r >= rows_dfs or c < 0 or c >= cols_dfs:
        return
    # 2. Cell is not the target color (it's a boundary or already filled)
    if grid[r][c] != target_color:
        return
    # 3. Cell is already the new color (optimization to prevent redundant calls)
    if grid[r][c] == new_color:
        return

    # Fill the current cell
    grid[r][c] = new_color

    # Recurse for neighbors (4-way: Up, Down, Left, Right)
    flood_fill_recursive(grid, r + 1, c, target_color, new_color) # Down
    flood_fill_recursive(grid, r - 1, c, target_color, new_color) # Up
    flood_fill_recursive(grid, r, c + 1, target_color, new_color) # Right
    flood_fill_recursive(grid, r, c - 1, target_color, new_color) # Left

# --- DFS Example Usage ---
print("Original Grid (DFS Example):")
print_grid(grid_dfs)

start_row_dfs, start_col_dfs = 1, 1
# Get the color at the starting point; this will be our target_color
original_color_at_start_dfs = grid_dfs[start_row_dfs][start_col_dfs]
fill_with_dfs = 'G' # Green

print(f"Filling from ({start_row_dfs},{start_col_dfs}) with '{fill_with_dfs}' (original '{original_color_at_start_dfs}')...")
flood_fill_recursive(grid_dfs, start_row_dfs, start_col_dfs, original_color_at_start_dfs, fill_with_dfs)

print("Grid after Recursive (DFS) Flood Fill:")
print_grid(grid_dfs)
```

#### Sample Output:

```output
Original Grid (DFS Example):
W W W W W W W
W B B W B B W
W B W W B W W
W B B W B B W
W W W W W W W
---------------
Filling from (1,1) with 'G' (original 'B')...
Grid after Recursive (DFS) Flood Fill:
W W W W W W W
W G G W B B W
W G W W B W W
W G G W B B W
W W W W W W W
---------------
```

Note how only the left-hand 'B' region was filled, as it was connected to the starting point `(1,1)`. The right-hand 'B' region, though identical in color, was separated by 'W' cells.

### 2. Iterative Flood Fill (BFS Approach)

The iterative approach uses a queue data structure to manage the cells to visit, mirroring a Breadth-First Search (BFS). This avoids the recursion depth limit issue by managing the state explicitly.

**How it works:**
1.  Initialize a queue and add the starting `(r, c)` cell to it.
2.  While the queue is not empty:
    *   Dequeue a cell `(curr_r, curr_c)`.
    *   If this cell is valid (within bounds, is the `target_color` and not yet the `new_color`):
        *   Change `grid[curr_r][curr_c]` to `new_color`.
        *   Enqueue all its valid (4-way) neighbors that are still the `target_color`.

**Pros:**
*   No stack overflow risk, making it suitable for very large grids/regions.
*   Generally preferred for production systems where robustness is key.

**Cons:**
*   Slightly more verbose due to explicit queue management.

#### Code Example: Iterative Flood Fill (4-way)

```python
from collections import deque

# Initial grid for BFS example (resetting to original state)
grid_bfs = [
    ['W', 'W', 'W', 'W', 'W', 'W', 'W'],
    ['W', 'B', 'B', 'W', 'B', 'B', 'W'],
    ['W', 'B', 'W', 'W', 'B', 'W', 'W'],
    ['W', 'B', 'B', 'W', 'B', 'B', 'W'],
    ['W', 'W', 'W', 'W', 'W', 'W', 'W']
]

rows_bfs = len(grid_bfs)
cols_bfs = len(grid_bfs[0])

def flood_fill_iterative(grid, r, c, target_color, new_color):
    """
    Performs a 4-way flood fill iteratively using BFS.
    grid: The 2D list representing the image/grid.
    r, c: Current row and column.
    target_color: The color to replace.
    new_color: The color to fill with.
    """
    # Initial checks: If start cell is already new_color or not target_color, do nothing.
    if grid[r][c] != target_color or target_color == new_color:
        return

    q = deque()
    q.append((r, c))
    grid[r][c] = new_color # Mark the starting cell as filled immediately

    # Directions for 4-way movement (Up, Down, Left, Right)
    dr = [-1, 1, 0, 0]
    dc = [0, 0, -1, 1]

    while q:
        curr_r, curr_c = q.popleft() # Dequeue a cell

        # Explore neighbors
        for i in range(4):
            next_r, next_c = curr_r + dr[i], curr_c + dc[i]

            # Check bounds and if neighbor is the target color
            if (0 <= next_r < rows_bfs and
                0 <= next_c < cols_bfs and
                grid[next_r][next_c] == target_color):
                grid[next_r][next_c] = new_color # Mark as filled before enqueuing
                q.append((next_r, next_c)) # Enqueue valid neighbor

# --- BFS Example Usage ---
print("Original Grid (BFS Example):")
print_grid(grid_bfs)

start_row_bfs, start_col_bfs = 1, 4 # Let's fill the right-hand 'B' region this time
original_color_at_start_bfs = grid_bfs[start_row_bfs][start_col_bfs]
fill_with_bfs = 'R' # Red

print(f"Filling from ({start_row_bfs},{start_col_bfs}) with '{fill_with_bfs}' (original '{original_color_at_start_bfs}')...")
flood_fill_iterative(grid_bfs, start_row_bfs, start_col_bfs, original_color_at_start_bfs, fill_with_bfs)

print("Grid after Iterative (BFS) Flood Fill:")
print_grid(grid_bfs)
```

#### Sample Output:

```output
Original Grid (BFS Example):
W W W W W W W
W B B W B B W
W B W W B W W
W B B W B B W
W W W W W W W
---------------
Filling from (1,4) with 'R' (original 'B')...
Grid after Iterative (BFS) Flood Fill:
W W W W W W W
W B B W R R W
W B W W R W W
W B B W R R W
W W W W W W W
---------------
```

This time, the right-hand 'B' region was filled with 'R', demonstrating the same functionality as DFS but via an iterative approach.

## Practical Considerations and Edge Cases

### Boundary Checks
Always ensure `r` and `c` indices are within the grid's dimensions (`0 <= r < rows` and `0 <= c < cols`). Failing to do so will result in `IndexError` or similar crashes. Both our examples correctly implement this.

### `target_color` vs. `new_color`
A subtle but important point:
*   If `target_color` is the *same* as `new_color`, the algorithm won't do anything because the condition `grid[r][c] == target_color` will fail after the first cell is changed, or the initial check `target_color == new_color` will return immediately. This is usually the desired behavior – no need to fill if it's already the target color!
*   If you *must* fill with the same color, you'd need a separate `visited` set to track cells that have been processed, rather than relying on the color change as the "visited" marker. However, for a typical flood fill operation, `target_color` and `new_color` are distinct.

### Four-way vs. Eight-way Flood Fill

Connectivity matters!
*   **4-way Connectivity (Manhattan Distance):** Only horizontal and vertical neighbors are considered. `(r±1, c)` and `(r, c±1)`. This is what we used in the examples so far.
*   **8-way Connectivity (Chebyshev Distance):** Includes diagonal neighbors in addition to horizontal and vertical. `(r±1, c)`, `(r, c±1)`, `(r±1, c±1)`, `(r±1, c∓1)`.

To implement 8-way flood fill, you just need to expand the list of `dr` (delta row) and `dc` (delta column) values that define the neighbor offsets.

#### Code Example: 4-way vs. 8-way Comparison

Let's use a grid where diagonal connection makes a difference.

```python
# Grid to demonstrate 4-way vs 8-way distinction
# X O X
# O X O
# X O X
# If we start at (0,0) (top-left 'X'), 4-way only fills itself.
# 8-way fills all 'X's.
grid_diagonal_test = [
    ['X', 'O', 'X', 'O', 'X'],
    ['O', 'X', 'O', 'X', 'O'],
    ['X', 'O', 'X', 'O', 'X'],
]

# Note: Using the recursive approach for simplicity in this demo.
# The same logic applies to BFS by extending dr/dc.
def flood_fill_generic_dfs(grid, r, c, target_color, new_color, is_8_way=False):
    """
    Generic DFS flood fill supporting both 4-way and 8-way.
    """
    rows, cols = len(grid), len(grid[0])

    if r < 0 or r >= rows or c < 0 or c >= cols:
        return
    if grid[r][c] != target_color:
        return
    if grid[r][c] == new_color: # Already filled, avoid redundant recursion
        return

    grid[r][c] = new_color

    # Directions for 4-way movement
    dr_4 = [-1, 1, 0, 0]
    dc_4 = [0, 0, -1, 1]

    # Directions for 8-way movement (including diagonals)
    dr_8 = [-1, -1, -1,  0, 0,  1, 1, 1]
    dc_8 = [-1,  0,  1, -1, 1, -1, 0, 1]

    directions_r = dr_8 if is_8_way else dr_4
    directions_c = dc_8 if is_8_way else dc_4

    for i in range(len(directions_r)):
        next_r, next_c = r + directions_r[i], c + directions_c[i]
        flood_fill_generic_dfs(grid, next_r, next_c, target_color, new_color, is_8_way)

# Make copies of the grid for comparison
grid_4way_fill_test = [row[:] for row in grid_diagonal_test]
grid_8way_fill_test = [row[:] for row in grid_diagonal_test]

print("Original Diagonal Grid:")
print_grid(grid_diagonal_test)

start_r, start_c = 0, 0 # Start at the top-left 'X'
original_color = grid_diagonal_test[start_r][start_c]

# --- 4-way Fill Test ---
new_color_4 = '4' # Mark with '4' for 4-way fill

print(f"Applying 4-way Flood Fill from ({start_r},{start_c}) with '{new_color_4}'...")
flood_fill_generic_dfs(grid_4way_fill_test, start_r, start_c, original_color, new_color_4, is_8_way=False)
print("Grid after 4-way fill:")
print_grid(grid_4way_fill_test)

# --- 8-way Fill Test ---
new_color_8 = '8' # Mark with '8' for 8-way fill

print(f"Applying 8-way Flood Fill from ({start_r},{start_c}) with '{new_color_8}'...")
flood_fill_generic_dfs(grid_8way_fill_test, start_r, start_c, original_color, new_color_8, is_8_way=True)
print("Grid after 8-way fill:")
print_grid(grid_8way_fill_test)
```

#### Sample Output:

```output
Original Diagonal Grid:
X O X O X
O X O X O
X O X O X
---------------
Applying 4-way Flood Fill from (0,0) with '4'...
Grid after 4-way fill:
4 O X O X
O X O X O
X O X O X
---------------
Applying 8-way Flood Fill from (0,0) with '8'...
Grid after 8-way fill:
8 O 8 O 8
O 8 O 8 O
8 O 8 O 8
---------------
```
Notice how the 4-way fill only changed the starting `X`, because its immediate (up/down/left/right) neighbors were `O`. But the 8-way fill correctly propagated through all diagonally connected `X`s.

### Performance (Time and Space Complexity)

*   **Time Complexity:** O(R * C), where R is the number of rows and C is the number of columns in the grid. In the worst case, the algorithm might visit every cell in the grid.
*   **Space Complexity:**
    *   **Recursive (DFS):** O(R * C) in the worst case (e.g., filling a long, winding path or the entire grid), due to the recursion stack depth.
    *   **Iterative (BFS):** O(R * C) in the worst case, as the queue might hold almost all cells simultaneously before they are processed.

## Real-World Applications

*   **Image Editors:** The most classic example is the "paint bucket" tool, which fills a contiguous region of a single color.
*   **Games:**
    *   **Minesweeper:** Used to reveal empty cells connected to a clicked empty cell.
    *   **Puzzle Games:** Identifying connected blocks, clearing regions, or checking for win conditions.
    *   **Map Generation:** Marking visited areas, pathfinding (though BFS/DFS are more general for pathfinding, flood fill can mark regions).
*   **Maze Solving:** Can be used to find all reachable cells from a starting point, essentially "solving" the maze by filling it.
*   **Connected Component Labeling:** In image processing or graph theory, flood fill can identify and label distinct connected regions.

## Conclusion

The Flood Fill algorithm is a cornerstone concept for any developer dealing with grid-based problems, image manipulation, or even basic graph traversals. Understanding its recursive (DFS) and iterative (BFS) implementations gives you powerful tools to solve a variety of challenges. While recursive is often more concise, the iterative approach offers robustness for large data sets by avoiding stack overflow issues.

By experimenting with the provided Python examples, you should now have a solid grasp of how Flood Fill works and how you can apply it in your own projects. Keep exploring and happy coding!
