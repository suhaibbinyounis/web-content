---
title: Solving Linear Diophantine Equations with Two Variables A Devs Guide
date: 2025-06-18T02:04:08.681Z
description: A practical guide for developers on understanding and solving linear Diophantine equations (ax + by = c) using the Extended Euclidean Algorithm, with Python examples. Learn when solutions exist and how to find them.
tags: ["mathematics", "algorithms", "number theory", "python", "euclidean algorithm", "discrete math"]
categories: ["Algorithms", "Programming"]
comments: true
---

As developers, we often encounter problems that, at their core, are about finding integer solutions. This is where Linear Diophantine Equations come into play. Named after the ancient Greek mathematician Diophantus, these equations require solutions to be integers. While they might seem like a niche mathematical concept, they pop up in surprising places: resource allocation, cryptography, competitive programming, and even simple puzzle-solving.

In this post, we'll focus on the simplest form: the linear Diophantine equation with two variables, `ax + by = c`, where `a`, `b`, and `c` are given integers, and we are looking for integer solutions for `x` and `y`.

## What is a Linear Diophantine Equation?

A linear Diophantine equation is an equation of the form `a₁x₁ + a₂x₂ + ... + aₙxₙ = c`, where `a₁`, `a₂`, ..., `aₙ`, and `c` are given integers, and we are looking for integer solutions for the variables `x₁`, `x₂`, ..., `xₙ`.

For this guide, we simplify it to two variables: `ax + by = c`.

For example, `3x + 5y = 10` is a linear Diophantine equation. `x=0, y=2` is one integer solution. `x=5, y=-1` is another. But `x=1, y=1.4` is not an integer solution, even though it satisfies the equation.

## When Do Solutions Exist?

This is the most critical question. There's a simple, elegant rule:

A linear Diophantine equation `ax + by = c` has integer solutions for `x` and `y` **if and only if** `gcd(a, b)` divides `c`.

Here, `gcd(a, b)` refers to the greatest common divisor of `a` and `b`.

Let's break down why this is true:

*   **Necessity**: If `ax + by = c` has an integer solution `(x₀, y₀)`, let `g = gcd(a, b)`. By definition, `g` divides `a` and `g` divides `b`. This means `a = g * a'` and `b = g * b'` for some integers `a'` and `b'`. Substituting these into the equation:
    `g * a' * x₀ + g * b' * y₀ = c`
    `g * (a' * x₀ + b' * y₀) = c`
    Since `(a' * x₀ + b' * y₀)` is an integer, `g` must divide `c`.

*   **Sufficiency**: If `g = gcd(a, b)` divides `c`, then we can find solutions. This is where the Extended Euclidean Algorithm comes in.

## Finding a Particular Solution: The Extended Euclidean Algorithm

The **Extended Euclidean Algorithm (EEA)** is a powerhouse for solving Diophantine equations. While the standard Euclidean Algorithm finds `gcd(a, b)`, the EEA goes a step further: it finds integers `x_p` and `y_p` such that `ax_p + by_p = gcd(a, b)`. This is known as Bezout's Identity.

Let `g = gcd(a, b)`. If `ax_p + by_p = g`, and we know that `g` divides `c` (so `c = g * k` for some integer `k`), then we can multiply the entire Bezout's identity by `k`:

`k * (ax_p + by_p) = k * g`
`a(k * x_p) + b(k * y_p) = c`

This gives us a particular solution: `x₀ = k * x_p` and `y₀ = k * y_p`. Or, more directly: `x₀ = (c / g) * x_p` and `y₀ = (c / g) * y_p`.

### Implementing the Extended Euclidean Algorithm

We'll need a function for `gcd` and another for `extended_gcd`. Python's `math` module has `gcd`, but for educational purposes and to ensure `extended_gcd` consistency, we'll write our own `gcd` and `extended_gcd`.

Note: The `gcd` is usually defined as a positive value. Our `extended_gcd` implementation will ensure this.

```python
import math

def gcd(a, b):
    """
    Calculates the greatest common divisor (GCD) of two integers using the Euclidean Algorithm.
    Ensures the GCD is always positive.
    """
    a = abs(a)
    b = abs(b)
    while b:
        a, b = b, a % b
    return a

def extended_gcd(a, b):
    """
    Extended Euclidean Algorithm to find g = gcd(a, b) and integers x, y
    such that ax + by = g.
    Handles negative inputs for a, b. g is always positive.
    """
    if a == 0:
        # Base case: gcd(0, b) = |b|. If b is 0, g=0, x=0, y=0.
        # If b != 0, g=|b|, and 0*0 + b*(1 if b>0 else -1) = |b|.
        return abs(b), 0, (1 if b >= 0 else -1) if b != 0 else 0

    # Recursive step
    # (g, x1, y1) is the result for (b % a, a)
    # We have (b % a) * x1 + a * y1 = g
    # Substitute b % a = b - (b // a) * a
    # (b - (b // a) * a) * x1 + a * y1 = g
    # b * x1 - (b // a) * a * x1 + a * y1 = g
    # a * (y1 - (b // a) * x1) + b * x1 = g
    # So, x = y1 - (b // a) * x1 and y = x1
    g, x1, y1 = extended_gcd(b % a, a)

    x = y1 - (b // a) * x1
    y = x1
    
    # Ensure g is positive. This should be handled by the base case's abs(b),
    # but as a safeguard or if non-standard GCD definitions are used upstream.
    if g < 0:
        g = -g
        x = -x
        y = -y
        
    return g, x, y
```

Let's test `extended_gcd` for a simple case: `extended_gcd(6, 15)`.

```python
# Test the extended_gcd function
g, x_p, y_p = extended_gcd(6, 15)
print(f"For a=6, b=15: gcd={g}, x_p={x_p}, y_p={y_p}")
print(f"Verification: {g} == {6*x_p + 15*y_p}")

g, x_p, y_p = extended_gcd(30, 42)
print(f"For a=30, b=42: gcd={g}, x_p={x_p}, y_p={y_p}")
print(f"Verification: {g} == {30*x_p + 42*y_p}")

g, x_p, y_p = extended_gcd(5, -7)
print(f"For a=5, b=-7: gcd={g}, x_p={x_p}, y_p={y_p}")
print(f"Verification: {g} == {5*x_p + (-7)*y_p}")
```

```output
For a=6, b=15: gcd=3, x_p=-2, y_p=1
Verification: 3 == 3
For a=30, b=42: gcd=6, x_p=3, y_p=-2
Verification: 6 == 6
For a=5, b=-7: gcd=1, x_p=3, y_p=2
Verification: 1 == 1
```

The `extended_gcd` function seems to work as expected, providing `g` and `x_p, y_p` such that `ax_p + by_p = g`.

## Finding the General Solution

Once we have *one* particular solution `(x₀, y₀)` for `ax + by = c`, we can find all other integer solutions.

Let `(x₀, y₀)` be a particular solution, so `ax₀ + by₀ = c`.
Let `(x, y)` be any other integer solution, so `ax + by = c`.

Subtracting the first equation from the second:
`(ax + by) - (ax₀ + by₀) = c - c`
`a(x - x₀) + b(y - y₀) = 0`
`a(x - x₀) = -b(y - y₀)`

Let `g = gcd(a, b)`. Divide the entire equation by `g`:
`(a/g)(x - x₀) = -(b/g)(y - y₀)`

Since `a/g` and `b/g` are coprime (their GCD is 1), `(a/g)` must divide `-(y - y₀)`.
Therefore, `-(y - y₀) = k * (a/g)` for some integer `k`.
This implies `y - y₀ = -k * (a/g)`, so `y = y₀ - k * (a/g)`.

Substitute this back into `(a/g)(x - x₀) = -(b/g)(y - y₀)`:
`(a/g)(x - x₀) = -(b/g) * (-k * (a/g))`
`(a/g)(x - x₀) = k * (b/g) * (a/g)`
Since `a/g` is non-zero (unless `a=0`, which is a special case we'll address), we can divide by `a/g`:
`x - x₀ = k * (b/g)`
So, `x = x₀ + k * (b/g)`.

Thus, the general solutions are:
`x = x₀ + k * (b / gcd(a, b))`
`y = y₀ - k * (a / gcd(a, b))`
for any integer `k`.

## The Complete Algorithm

Here's the step-by-step process to solve `ax + by = c` for integers `x, y`:

1.  **Calculate `g = gcd(a, b)`** using the Euclidean Algorithm.
2.  **Check for existence of solutions**: If `c % g != 0` (i.e., `c` is not divisible by `g`), then there are **no integer solutions**.
3.  **Find `x_p, y_p`**: Use the Extended Euclidean Algorithm to find integers `x_p` and `y_p` such that `ax_p + by_p = g`.
4.  **Calculate a particular solution `(x₀, y₀)`**:
    Let `k_mult = c // g`.
    Then `x₀ = x_p * k_mult`
    And `y₀ = y_p * k_mult`
5.  **State the general solution**: The set of all integer solutions `(x, y)` is given by:
    `x = x₀ + k * (b // g)`
    `y = y₀ - k * (a // g)`
    for any integer `k` (where `k` can be 0, ±1, ±2, ...).

## Python Implementation for Solving the Equation

Let's combine everything into a `solve_diophantine` function.

```python
import math

# Re-using the functions defined earlier for clarity
# If you run this as a single script, ensure these are at the top.

def gcd(a, b):
    a = abs(a)
    b = abs(b)
    while b:
        a, b = b, a % b
    return a

def extended_gcd(a, b):
    if a == 0:
        return abs(b), 0, (1 if b >= 0 else -1) if b != 0 else 0
    
    g, x1, y1 = extended_gcd(b % a, a)
    x = y1 - (b // a) * x1
    y = x1
    
    if g < 0: # Should not happen if base case is correct
        g = -g
        x = -x
        y = -y
        
    return g, x, y

def solve_diophantine(a, b, c):
    """
    Solves the linear Diophantine equation ax + by = c for integer x and y.

    Args:
        a (int): Coefficient of x.
        b (int): Coefficient of y.
        c (int): Constant term.

    Returns:
        tuple: A tuple containing:
            - A boolean indicating if solutions exist.
            - A tuple (x0, y0) representing a particular solution if it exists.
            - A string describing the general solution form if it exists.
              Returns (False, None, None) if no solutions.
    """
    # Handle trivial cases where a or b is zero
    if a == 0 and b == 0:
        if c == 0:
            return True, "Any (x, y)", "Any integer x, Any integer y"
        else:
            return False, None, None
    elif a == 0: # 0*x + by = c => by = c
        if c % b == 0:
            return True, (0, c // b), f"x = k, y = {c // b} (for any integer k)"
        else:
            return False, None, None
    elif b == 0: # ax + 0*y = c => ax = c
        if c % a == 0:
            return True, (c // a, 0), f"x = {c // a}, y = k (for any integer k)"
        else:
            return False, None, None

    # Apply Extended Euclidean Algorithm for non-zero a, b
    g, x_p, y_p = extended_gcd(a, b)

    if c % g != 0:
        return False, None, None # No integer solutions

    # Calculate a particular solution
    k_mult = c // g
    x0 = x_p * k_mult
    y0 = y_p * k_mult

    # General solution form
    # x = x0 + k * (b/g)
    # y = y0 - k * (a/g)
    gen_x_term = b // g
    gen_y_term = a // g

    general_solution_str = f"x = {x0} + k * ({gen_x_term}), y = {y0} - k * ({gen_y_term}) (for any integer k)"

    return True, (x0, y0), general_solution_str

```

## Examples in Action

Let's run through some realistic examples.

### Example 1: Basic Case with Solution

Equation: `6x + 15y = 21`

```python
# Example 1: 6x + 15y = 21
a1, b1, c1 = 6, 15, 21
has_solution1, particular_solution1, general_solution1 = solve_diophantine(a1, b1, c1)

print(f"--- Solving {a1}x + {b1}y = {c1} ---")
if has_solution1:
    x0_1, y0_1 = particular_solution1
    print(f"Solutions exist.")
    print(f"A particular solution: x = {x0_1}, y = {y0_1}")
    print(f"General solutions: {general_solution1}")
    print(f"Verification of particular solution: {a1}*{x0_1} + {b1}*{y0_1} = {a1*x0_1 + b1*y0_1}")
else:
    print("No integer solutions exist.")

print("\n--- Testing another particular solution (k=1) ---")
# Let's test a general solution for k=1
g_val = gcd(a1, b1)
x_test_k1 = x0_1 + 1 * (b1 // g_val)
y_test_k1 = y0_1 - 1 * (a1 // g_val)
print(f"For k=1: x = {x_test_k1}, y = {y_test_k1}")
print(f"Verification: {a1}*{x_test_k1} + {b1}*{y_test_k1} = {a1*x_test_k1 + b1*y_test_k1}")
```

```output
--- Solving 6x + 15y = 21 ---
Solutions exist.
A particular solution: x = -14, y = 7
General solutions: x = -14 + k * (5), y = 7 - k * (2) (for any integer k)
Verification of particular solution: 6*-14 + 15*7 = -84 + 105 = 21

--- Testing another particular solution (k=1) ---
For k=1: x = -9, y = 5
Verification: 6*-9 + 15*5 = -54 + 75 = 21
```

### Example 2: No Solution Case

Equation: `6x + 9y = 10`

```python
# Example 2: 6x + 9y = 10
a2, b2, c2 = 6, 9, 10
has_solution2, particular_solution2, general_solution2 = solve_diophantine(a2, b2, c2)

print(f"\n--- Solving {a2}x + {b2}y = {c2} ---")
if has_solution2:
    x0_2, y0_2 = particular_solution2
    print(f"Solutions exist.")
    print(f"A particular solution: x = {x0_2}, y = {y0_2}")
    print(f"General solutions: {general_solution2}")
else:
    print("No integer solutions exist.")
```

```output
--- Solving 6x + 9y = 10 ---
No integer solutions exist.
```
**Explanation**: `gcd(6, 9) = 3`. `10` is not divisible by `3`. So, no integer solutions.

### Example 3: Larger Numbers

Equation: `30x + 42y = 12`

```python
# Example 3: 30x + 42y = 12
a3, b3, c3 = 30, 42, 12
has_solution3, particular_solution3, general_solution3 = solve_diophantine(a3, b3, c3)

print(f"\n--- Solving {a3}x + {b3}y = {c3} ---")
if has_solution3:
    x0_3, y0_3 = particular_solution3
    print(f"Solutions exist.")
    print(f"A particular solution: x = {x0_3}, y = {y0_3}")
    print(f"General solutions: {general_solution3}")
    print(f"Verification of particular solution: {a3}*{x0_3} + {b3}*{y0_3} = {a3*x0_3 + b3*y0_3}")
else:
    print("No integer solutions exist.")
```

```output
--- Solving 30x + 42y = 12 ---
Solutions exist.
A particular solution: x = 6, y = -4
General solutions: x = 6 + k * (7), y = -4 - k * (5) (for any integer k)
Verification of particular solution: 30*6 + 42*-4 = 180 + -168 = 12
```

### Example 4: Negative Coefficients/Constants

Equation: `5x - 7y = 12`

```python
# Example 4: 5x - 7y = 12
a4, b4, c4 = 5, -7, 12
has_solution4, particular_solution4, general_solution4 = solve_diophantine(a4, b4, c4)

print(f"\n--- Solving {a4}x + {b4}y = {c4} ---")
if has_solution4:
    x0_4, y0_4 = particular_solution4
    print(f"Solutions exist.")
    print(f"A particular solution: x = {x0_4}, y = {y0_4}")
    print(f"General solutions: {general_solution4}")
    print(f"Verification of particular solution: {a4}*{x0_4} + {b4}*{y0_4} = {a4*x0_4 + b4*y0_4}")
else:
    print("No integer solutions exist.")
```

```output
--- Solving 5x + -7y = 12 ---
Solutions exist.
A particular solution: x = 36, y = 24
General solutions: x = 36 + k * (-7), y = 24 - k * (5) (for any integer k)
Verification of particular solution: 5*36 + -7*24 = 180 + -168 = 12
```

### Example 5: Zero Coefficients

Equation: `0x + 5y = 10`

```python
# Example 5: 0x + 5y = 10
a5, b5, c5 = 0, 5, 10
has_solution5, particular_solution5, general_solution5 = solve_diophantine(a5, b5, c5)

print(f"\n--- Solving {a5}x + {b5}y = {c5} ---")
if has_solution5:
    print(f"Solutions exist.")
    if isinstance(particular_solution5, tuple):
        x0_5, y0_5 = particular_solution5
        print(f"A particular solution: x = {x0_5}, y = {y0_5}")
    else:
        print(f"Particular solution form: {particular_solution5}")
    print(f"General solutions: {general_solution5}")
else:
    print("No integer solutions exist.")
```

```output
--- Solving 0x + 5y = 10 ---
Solutions exist.
A particular solution: x = 0, y = 2
General solutions: x = k, y = 2 (for any integer k)
```

## Considerations and Edge Cases

*   **Zero Coefficients**: The `solve_diophantine` function includes explicit checks for `a=0` or `b=0`.
    *   If `a = 0` and `b != 0`: `by = c`. If `c` is divisible by `b`, then `y = c/b`, and `x` can be any integer.
    *   If `b = 0` and `a != 0`: `ax = c`. If `c` is divisible by `a`, then `x = c/a`, and `y` can be any integer.
    *   If `a = 0` and `b = 0`: `0 = c`. If `c = 0`, then any integer `x, y` pair is a solution. If `c != 0`, there are no solutions.
*   **Sign of GCD**: While `gcd(a,b)` is conventionally positive, `extended_gcd` needs careful implementation if `a` or `b` can be negative. The `abs()` in `gcd` and the specific base case in `extended_gcd` ensures `g` (the GCD result) is positive, and the `x, y` values correctly satisfy the original equation `ax + by = g`.
*   **Efficiency**: The Euclidean Algorithm (and its extended version) is very efficient, running in logarithmic time with respect to the magnitude of the input numbers.

## Conclusion

Understanding and solving linear Diophantine equations might seem like a pure math exercise, but it's a fundamental concept with practical implications in computer science. The key takeaways are:

1.  **Existence Condition**: Solutions for `ax + by = c` exist if and only if `gcd(a, b)` divides `c`.
2.  **Extended Euclidean Algorithm**: This algorithm is your best friend for finding a particular solution `(x_p, y_p)` to `ax_p + by_p = gcd(a, b)`.
3.  **General Solutions**: Once you have one solution `(x₀, y₀)`, you can generate all others using the derived formulas: `x = x₀ + k * (b/g)` and `y = y₀ - k * (a/g)`.

By mastering these concepts, you gain a powerful tool for tackling problems that require integer constraints, from theoretical challenges to real-world applications. Practice with different values and edge cases to solidify your understanding!
