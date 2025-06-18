---
title: Solving the N-Queens Problem with Backtracking
date: 2025-06-18T02:04:08.681Z
description: "A comprehensive guide to understanding and solving the classic N-Queens problem using the backtracking algorithm in Python, with detailed code examples and explanations."
tags: [Algorithms, Backtracking, Python, Data Structures, Problem Solving, Computer Science]
categories: [Programming, Algorithms]
comments: true
---

The N-Queens problem is a classic puzzle in computer science that serves as an excellent introduction to algorithmic techniques like backtracking. It's a fundamental problem often used to illustrate how to navigate a search space systematically, pruning branches that don't lead to a solution.

Let's dive in.

## What is the N-Queens Problem?

The N-Queens problem challenges us to place `N` non-attacking queens on an `N × N` chessboard. "Non-attacking" means that no two queens can threaten each other.

What constitutes an attack?
1.  **Same Row**: Two queens in the same horizontal line.
2.  **Same Column**: Two queens in the same vertical line.
3.  **Same Diagonal**: Two queens on the same diagonal line (either top-left to bottom-right or top-right to bottom-left).

The goal is to find all possible distinct configurations of queens that satisfy these conditions.

For example, for `N=1`, there's 1 solution. For `N=2` and `N=3`, there are no solutions. For `N=4`, there are 2 solutions.

## Why is it Interesting?

Beyond being a fun puzzle, the N-Queens problem is a poster child for:
*   **Backtracking Algorithms**: It perfectly demonstrates the "explore and retreat" mechanism.
*   **Constraint Satisfaction**: We're trying to find a configuration that satisfies a set of constraints.
*   **State-Space Search**: It involves navigating a large number of possibilities efficiently.

## Core Idea: Backtracking

Backtracking is an algorithmic paradigm for solving problems, typically constraint-satisfaction problems, by incrementally building a solution and abandoning (backtracking from) any partial solutions that fail to satisfy the problem's constraints.

Here's how it applies to N-Queens:

1.  **Place a Queen**: Start by placing a queen in the first row, at some column.
2.  **Check Safety**: After placing it, check if it's "safe" – does it attack any previously placed queens?
3.  **Recurse**: If it's safe, move to the next row and repeat the process.
4.  **Backtrack**: If no safe column is found in the current row (meaning the previous placement led to a dead end), "undo" the last placement and try a different column in the *previous* row.
5.  **Base Case**: If all `N` queens are successfully placed (i.e., we've reached row `N`), we've found a valid solution.

This recursive exploration, combined with the "safety check" (which prunes invalid paths early), is the essence of backtracking.

## Algorithm Breakdown

To implement this, we need a way to represent the board and efficiently check for conflicts.

### Board Representation

We only need to store the column position for each queen. Since we place one queen per row, an array (or list in Python) where `board[row] = col` is perfect. This inherently handles the "no two queens in the same row" constraint.

For an `N x N` board, `board = [0] * N` could represent the board, where `board[i]` stores the column where the queen in row `i` is placed.

### `is_safe(board, row, col)` Function

This crucial function checks if placing a queen at `(row, col)` conflicts with any queens already placed in `0` to `row-1`.

We need to check:
*   **Column Conflict**: Is `col` already occupied by a queen in any previous row? This means `board[i] == col` for any `i < row`.
*   **Diagonal Conflict**: Are there any queens on the same diagonals?
    *   A queen at `(r1, c1)` and another at `(r2, c2)` are on the same main diagonal if `r1 - c1 == r2 - c2`.
    *   They are on the same anti-diagonal if `r1 + c1 == r2 + c2`.
    *   A simpler way to check this: `abs(r1 - r2) == abs(c1 - c2)`. For our `is_safe` function, this translates to `abs(i - row) == abs(board[i] - col)` for any `i < row`.

Let's look at an example implementation for `is_safe`.

```python
def is_safe(board, row, col):
    """
    Checks if placing a queen at (row, col) is safe.
    'board' stores the column position for the queen in each row.
    """
    # Check this column in previous rows
    for i in range(row):
        if board[i] == col:
            return False

    # Check upper-left diagonal
    # (row - i) == (col - board[i])  => row - col == i - board[i]
    for i in range(row):
        if board[i] == col - (row - i): # board[i] == col - delta_row
            return False

    # Check upper-right diagonal
    # (row - i) == -(col - board[i]) => row + col == i + board[i]
    for i in range(row):
        if board[i] == col + (row - i): # board[i] == col + delta_row
            return False

    return True

# Example Usage: (Conceptual)
# Imagine board = [1, 3, 0, -1] for N=4, meaning:
# Q at (0,1), Q at (1,3), Q at (2,0), and we're trying to place at (3,col)
# Let's test is_safe(board, 3, 2)
# is_safe would check:
# For i=0 (board[0]=1):
#   col check: board[0](1) != 2 (ok)
#   UL diag: abs(0-3) == abs(1-2) => 3 == 1 (False, ok)
#   UR diag: abs(0-3) == abs(1-2) => 3 == 1 (False, ok)
# For i=1 (board[1]=3):
#   col check: board[1](3) != 2 (ok)
#   UL diag: abs(1-3) == abs(3-2) => 2 == 1 (False, ok)
#   UR diag: abs(1-3) == abs(3-2) => 2 == 1 (False, ok)
# For i=2 (board[2]=0):
#   col check: board[2](0) != 2 (ok)
#   UL diag: abs(2-3) == abs(0-2) => 1 == 2 (False, ok)
#   UR diag: abs(2-3) == abs(0-2) => 1 == 2 (False, ok)
# So, placing at (3,2) would be safe with this partial board.
```

The diagonal checks can be simplified to a single loop using `abs(row1 - row2) == abs(col1 - col2)`.

```python
def is_safe_simplified(board, row, col):
    """
    Checks if placing a queen at (row, col) is safe.
    'board' stores the column position for the queen in each row.
    This version uses the simplified diagonal check.
    """
    for i in range(row):
        # Check column conflict
        if board[i] == col:
            return False
        # Check diagonal conflict (main diagonal and anti-diagonal)
        # abs(row1 - row2) == abs(col1 - col2)
        if abs(i - row) == abs(board[i] - col):
            return False
    return True
```
This simplified `is_safe` is more elegant and easier to reason about. We'll use this one.

### `solve_n_queens(row, board, n, solutions)` Function

This is the recursive backtracking function.

```python
def solve_n_queens_recursive(row, board, n, solutions):
    """
    Recursive function to find all N-Queens solutions.
    :param row: The current row we are trying to place a queen in.
    :param board: A list representing the board state (board[i] = col for queen in row i).
    :param n: The size of the chessboard.
    :param solutions: A list to store all valid board configurations found.
    """
    # Base Case: If all queens are placed (we've successfully placed a queen in row n-1)
    if row == n:
        # A solution is found. Add a copy of the current board state to solutions.
        # We need a copy because 'board' is modified by subsequent recursive calls.
        solutions.append(list(board))
        return

    # Recursive Step: Try placing a queen in each column of the current row
    for col in range(n):
        # Check if it's safe to place a queen at (row, col)
        if is_safe_simplified(board, row, col):
            # Place the queen
            board[row] = col

            # Recurse for the next row
            solve_n_queens_recursive(row + 1, board, n, solutions)

            # Backtrack: No explicit "undo" is needed for `board[row] = col`
            # because when the loop iterates to the next `col`, or when the
            # function returns, the value for `board[row]` is implicitly
            # overwritten in the next iteration for the current `row`,
            # or discarded as the stack unwinds. However, if 'board' were
            # a global state or class member, we'd need to reset it.
            # Here, the changes are localized to the current recursion path.
            # Note: For array-like objects modified in place, typically you *do*
            # need to backtrack. In this specific case, `board[row] = col`
            # for the current `row` will be overwritten by subsequent calls
            # to `solve_n_queens_recursive` for this `row`, or effectively
            # "undone" as the stack unwinds and previous `row`'s context restores.
            # To be explicit and robust for all backtracking scenarios:
            # board[row] = -1 # Or some sentinel value indicating no queen.
            # However, for this problem, the re-assignment in the loop is sufficient.
            # Let's stick with the simplest (and common) form for N-Queens.
```

**Note**: The backtracking step in N-Queens often confuses beginners. Because we assign `board[row] = col` for a specific `row`, when the loop for `col` finishes (either because `solve_n_queens_recursive(row + 1, ...)` returned, or `is_safe` was false for all columns), that `board[row]` value is effectively "undone" for the *next* iteration of `col` in the *current* `row`'s context, or when the function returns and the caller's `board[row-1]` context is restored. No explicit removal is strictly necessary for this specific problem and representation.

### Putting It All Together (Python)

Let's combine these into a complete, runnable script. We'll also add a helper to print the solutions in a readable chessboard format.

```python
#!/usr/bin/env python3

def is_safe(board, row, col):
    """
    Checks if placing a queen at (row, col) is safe from previously placed queens.
    'board' is a list where board[i] stores the column of the queen in row i.
    """
    for i in range(row):
        # Check column conflict
        if board[i] == col:
            return False
        # Check diagonal conflict: |row1 - row2| == |col1 - col2|
        if abs(i - row) == abs(board[i] - col):
            return False
    return True

def solve_n_queens_recursive(row, board, n, solutions):
    """
    Recursive function to find all N-Queens solutions.
    :param row: The current row being considered for placing a queen.
    :param board: Current state of the board (list of column positions).
    :param n: The size of the chessboard (N x N).
    :param solutions: List to store all found valid board configurations.
    """
    # Base case: All queens have been successfully placed (we've filled up to row n-1)
    if row == n:
        # A solution is found. Add a copy of the current board configuration.
        solutions.append(list(board))
        return

    # Recursive step: Try placing a queen in each column of the current 'row'
    for col in range(n):
        if is_safe(board, row, col):
            # Place the queen
            board[row] = col
            # Recurse for the next row
            solve_n_queens_recursive(row + 1, board, n, solutions)
            # Backtrack: No explicit undo for 'board[row]' needed here as the
            # loop's next iteration will overwrite it, or the function's return
            # will effectively restore the caller's context for previous rows.

def solve_n_queens(n):
    """
    Main function to initiate the N-Queens problem solving.
    :param n: The size of the chessboard.
    :return: A list of all solutions, where each solution is a list
             representing queen positions (e.g., [1, 3, 0, 2] for N=4).
    """
    if n <= 0:
        return []
    
    # Initialize the board with a sentinel value (e.g., -1)
    # This board will store the column index for the queen in each row.
    board = [-1] * n
    solutions = []
    
    solve_n_queens_recursive(0, board, n, solutions)
    return solutions

def print_solution(board, n):
    """
    Prints a single N-Queens solution in a visual chessboard format.
    'board' is a list of column positions.
    """
    for row in range(n):
        line = ["." for _ in range(n)]
        line[board[row]] = "Q"
        print(" ".join(line))
    print() # Add a blank line for separation

# --- Main execution ---
if __name__ == "__main__":
    # Example 1: N = 4
    print("--- Solving N-Queens for N = 4 ---")
    n4_solutions = solve_n_queens(4)
    print(f"Found {len(n4_solutions)} solutions for N=4:")
    for i, sol in enumerate(n4_solutions):
        print(f"Solution {i+1}: {sol}")
        print_solution(sol, 4)

    # Example 2: N = 8
    print("\n--- Solving N-Queens for N = 8 ---")
    n8_solutions = solve_n_queens(8)
    print(f"Found {len(n8_solutions)} solutions for N=8.")
    # For N=8, printing all solutions would be too verbose,
    # so let's just print the first one.
    if n8_solutions:
        print("\nFirst solution for N=8:")
        print_solution(n8_solutions[0], 8)
    else:
        print("No solutions found for N=8.")

    # Example 3: N = 1 (trivial)
    print("\n--- Solving N-Queens for N = 1 ---")
    n1_solutions = solve_n_queens(1)
    print(f"Found {len(n1_solutions)} solutions for N=1:")
    for i, sol in enumerate(n1_solutions):
        print(f"Solution {i+1}: {sol}")
        print_solution(sol, 1)

    # Example 4: N = 2 (no solutions)
    print("\n--- Solving N-Queens for N = 2 ---")
    n2_solutions = solve_n_queens(2)
    print(f"Found {len(n2_solutions)} solutions for N=2.")
```

### Sample Output

To run the script, save it as `n_queens.py` and execute from your terminal:

```bash
python3 n_queens.py
```

```output
--- Solving N-Queens for N = 4 ---
Found 2 solutions for N=4:
Solution 1: [1, 3, 0, 2]
. Q . .
. . . Q
Q . . .
. . Q .

Solution 2: [2, 0, 3, 1]
. . Q .
Q . . .
. . . Q
. Q . .

--- Solving N-Queens for N = 8 ---
Found 92 solutions for N=8.

First solution for N=8:
Q . . . . . . .
. . . . Q . . .
. . . . . . . Q
. . . . . Q . .
. . Q . . . . .
. . . . . . Q .
. Q . . . . . .
. . . Q . . . .

--- Solving N-Queens for N = 1 ---
Found 1 solutions for N=1:
Solution 1: [0]
Q 

--- Solving N-Queens for N = 2 ---
Found 0 solutions for N=2.
```

As you can see, for N=4, we get the two expected solutions. For N=8, there are 92 solutions, and we print one of them. N=1 gives one trivial solution, and N=2 correctly yields no solutions.

## Analyzing Complexity

### Time Complexity

The N-Queens problem is a classic example of how pruning (via the `is_safe` function) significantly reduces the search space compared to a brute-force approach.

*   **Brute-Force**: Without any checks, we'd try `N!` permutations of queen placements, and for each, we'd check `O(N^2)` for conflicts. This is roughly `O(N! * N^2)`.
*   **Backtracking**: The actual time complexity is much better. In the worst case, without any pruning, it could still approach `O(N!)` if `is_safe` always returns true until the very end. However, `is_safe` quickly prunes invalid branches. The practical complexity is hard to express with a simple Big-O notation due to the pruning effect, but it's significantly less than `N!`. It's closer to `O(N^N)` in terms of states *visited*, but *much* better in practice.
    A tighter upper bound for some variations is around `O(N!)`. For N-Queens, it's often described as `O(N^N)` in the absolute worst theoretical case (trying all positions for all queens) but practically much faster due to the aggressive pruning.
    For typical `N` values (up to ~15-20), it's very fast.

### Space Complexity

*   The `board` list uses `O(N)` space to store the queen positions.
*   The recursion depth goes up to `N`. This means the call stack also uses `O(N)` space.
*   The `solutions` list stores all found solutions. In the worst case, the number of solutions can be large (e.g., 92 for N=8), so storing all of them can take `O(Number_of_Solutions * N)` space. If you only need to count solutions or print them one by one, this can be reduced.

Overall, the auxiliary space complexity for the algorithm itself (excluding storing all solutions) is `O(N)`.

## Beyond the Basics (Briefly)

While the backtracking approach shown is standard and efficient for typical N values:

*   **Bit Manipulation**: For highly optimized competitive programming solutions, the `is_safe` checks (especially for diagonals) can be implemented using bitmasks. This can provide a significant speedup by leveraging bitwise operations on integers to represent occupied rows, columns, and diagonals, which are much faster than array lookups and arithmetic operations.
*   **Optimized Search**: For very large `N`, other techniques like constraint programming libraries or more advanced search algorithms might be employed, though N-Queens is primarily an educational problem for backtracking.

## Real-world Relevance

The N-Queens problem, while seemingly an abstract puzzle, embodies core principles applicable to many real-world problems:

*   **Resource Allocation**: Assigning limited resources (e.g., people, machines, frequencies) to tasks while avoiding conflicts.
*   **Scheduling**: Creating schedules (e.g., for classes, meetings, flights) where events don't overlap or violate constraints.
*   **AI and Game Theory**: Pathfinding, decision-making in games, or constraint satisfaction in expert systems.
*   **Combinatorial Optimization**: Finding the "best" solution among a vast number of possibilities under given constraints.

Backtracking itself is a fundamental technique for problems involving permutations, combinations, and general search where you build a solution step-by-step and need to revert choices if they lead to an invalid state.

## Conclusion

The N-Queens problem is a beautiful illustration of how backtracking systematically explores a search space. By implementing an `is_safe` function to prune invalid paths early, we transform a potentially intractable problem (brute-force `N! * N^2`) into a manageable one. Understanding this problem and its solution is a significant step towards mastering recursive algorithms and constraint satisfaction in programming.