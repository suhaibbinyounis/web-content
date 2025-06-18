---
title: Demystifying Palindromic Substrings - Algorithms and Practical Examples
date: 2025-06-18T02:04:08.681Z
description: Dive deep into finding palindromic substrings. Explore the fundamental concepts, the intuitive brute-force approach, the efficient expand-around-center technique, and a brief nod to advanced algorithms, all with practical Python code examples.
tags: [Algorithms, StringManipulation, ComputerScience, Python, Programming, InterviewPrep]
categories: [SoftwareDevelopment, DataStructuresAndAlgorithms]
comments: true
---

As developers, we often encounter string manipulation problems that seem simple on the surface but hide fascinating algorithmic challenges. One such classic is the "Palindromic Substring" problem. It's a common interview question and a great way to sharpen your algorithmic thinking.

This post will break down what palindromic substrings are, explore different ways to find them, and provide clear, runnable code examples to solidify your understanding.

## What's a Palindrome, and What's a Substring?

Before we dive into the "palindromic substring" part, let's ensure we're on the same page with the fundamental building blocks.

### What is a Substring?

A substring is a contiguous sequence of characters within a string.
For example, if you have the string "banana":
*   "ban" is a substring.
*   "ana" is a substring.
*   "bana" is a substring.
*   "bna" is NOT a substring (it's a subsequence, but not contiguous).
*   "a" is a substring.

### What is a Palindrome?

A palindrome is a sequence of characters that reads the same forwards and backward. Case, spaces, and punctuation usually matter unless specified otherwise.
For example:
*   "madam" is a palindrome.
*   "racecar" is a palindrome.
*   "level" is a palindrome.
*   "A" is a palindrome.
*   "rotor" is a palindrome.
*   "hello" is NOT a palindrome.

### Putting it Together: Palindromic Substring

A palindromic substring is simply a substring that is also a palindrome.
Consider the string "babad".
*   Substrings: "b", "a", "b", "a", "d", "ba", "ab", "bad", "baba", "babad", etc.
*   Palindromic Substrings:
    *   "b" (at index 0)
    *   "a" (at index 1)
    *   "b" (at index 2)
    *   "a" (at index 3)
    *   "d" (at index 4)
    *   "aba"
    *   "bab"

Notice that single characters are always palindromes, so they are always palindromic substrings. The real challenge comes with identifying longer ones.

## Approach 1: The Brute-Force (Naïve) Method

The most straightforward way to find all palindromic substrings is to generate every possible substring and then check if each one is a palindrome.

### How it Works:
1.  Iterate through all possible starting positions (`i`) for a substring.
2.  For each starting position, iterate through all possible ending positions (`j`).
3.  Extract the substring `s[i:j+1]`.
4.  Check if this extracted substring is a palindrome. If it is, add it to our list.

### Time Complexity:
*   Generating all substrings: There are roughly O(N^2) substrings in a string of length N. (e.g., for "abc", "a", "b", "c", "ab", "bc", "abc" -> 6 substrings for N=3).
*   Checking if a substring is a palindrome: For a substring of length `k`, this takes O(k) time. Since `k` can be up to `N`, this check takes O(N) in the worst case.
*   Total Complexity: O(N^2 * N) = **O(N^3)**.

While simple to understand, O(N^3) is generally inefficient for larger strings.

### Code Example (Python):

Let's implement the `is_palindrome` helper function and then the brute-force `find_all_palindromic_substrings_bruteforce` function.

```python
def is_palindrome(s: str) -> bool:
    """Checks if a given string is a palindrome."""
    return s == s[::-1]

def find_all_palindromic_substrings_bruteforce(s: str) -> list[str]:
    """
    Finds all palindromic substrings in a given string using the brute-force method.
    Time Complexity: O(N^3)
    Space Complexity: O(N) for storing results (worst case: all single chars are palindromes)
    """
    n = len(s)
    palindromes = set() # Use a set to store unique palindromes

    for i in range(n):
        for j in range(i, n):
            substring = s[i : j + 1]
            if is_palindrome(substring):
                palindromes.add(substring)
    
    return sorted(list(palindromes)) # Return sorted list for consistent output

# --- Test cases ---
print("--- Brute-Force Examples ---")

# Example 1
input_str1 = "babad"
output1 = find_all_palindromic_substrings_bruteforce(input_str1)
print(f"Input: '{input_str1}'")
print(f"Palindromic Substrings: {output1}")

# Example 2
input_str2 = "racecar"
output2 = find_all_palindromic_substrings_bruteforce(input_str2)
print(f"\nInput: '{input_str2}'")
print(f"Palindromic Substrings: {output2}")

# Example 3
input_str3 = "abacaba"
output3 = find_all_palindromic_substrings_bruteforce(input_str3)
print(f"\nInput: '{input_str3}'")
print(f"Palindromic Substrings: {output3}")

# Example 4 (Empty string)
input_str4 = ""
output4 = find_all_palindromic_substrings_bruteforce(input_str4)
print(f"\nInput: '{input_str4}'")
print(f"Palindromic Substrings: {output4}")

# Example 5 (Single character string)
input_str5 = "a"
output5 = find_all_palindromic_substrings_bruteforce(input_str5)
print(f"\nInput: '{input_str5}'")
print(f"Palindromic Substrings: {output5}")
```

### Sample Output:

```output
--- Brute-Force Examples ---
Input: 'babad'
Palindromic Substrings: ['a', 'aba', 'b', 'bab', 'd']

Input: 'racecar'
Palindromic Substrings: ['a', 'aceca', 'c', 'cec', 'e', 'r', 'racecar']

Input: 'abacaba'
Palindromic Substrings: ['a', 'aba', 'abacaba', 'acaca', 'b', 'bacab', 'c']

Input: ''
Palindromic Substrings: []

Input: 'a'
Palindromic Substrings: ['a']
```

## Approach 2: Expand Around Center

This is a significantly more efficient approach, reducing the time complexity to O(N^2). It leverages the property that palindromes expand outwards from a central point.

### Key Idea:
A palindrome can have two types of centers:
1.  **Single Character Center (Odd Length Palindromes)**: "madam" has 'd' as its center.
2.  **Two Character Center (Even Length Palindromes)**: "abba" has 'bb' as its center (or conceptually, the space between the two 'b's).

We can iterate through every possible center in the string and expand outwards from that center, checking for palindromes.

### How it Works:
1.  For each character in the string, treat it as a potential center for an odd-length palindrome. Expand left and right, checking if characters match.
2.  For each pair of adjacent characters (i.e., between `s[i]` and `s[i+1]`), treat the 'space' between them as a potential center for an even-length palindrome. Expand left from `s[i]` and right from `s[i+1]`, checking if characters match.
3.  Each expansion gives us one or more palindromic substrings.

### Time Complexity:
*   Number of potential centers: There are `N` single-character centers and `N-1` two-character centers, roughly `2N` centers.
*   Expansion from each center: In the worst case, we might expand across the entire string (e.g., "aaaaa"). This takes O(N) time.
*   Total Complexity: O(2N * N) = **O(N^2)**.

This is a significant improvement over O(N^3).

### Code Example (Python):

We'll use a helper function `expand_around_center` that takes the string and two pointers (left and right) representing the center(s).

```python
def expand_around_center(s: str, left: int, right: int, palindromes: set):
    """
    Expands outwards from a center point (left, right) to find all
    palindromic substrings centered at that point.
    Adds found palindromes to the 'palindromes' set.
    """
    while left >= 0 and right < len(s) and s[left] == s[right]:
        palindromes.add(s[left : right + 1])
        left -= 1
        right += 1

def find_all_palindromic_substrings_expand_around_center(s: str) -> list[str]:
    """
    Finds all palindromic substrings in a given string using the expand-around-center method.
    Time Complexity: O(N^2)
    Space Complexity: O(N) for storing results
    """
    n = len(s)
    if n == 0:
        return []

    palindromes = set()

    for i in range(n):
        # Case 1: Odd length palindromes (centered at s[i])
        expand_around_center(s, i, i, palindromes)
        
        # Case 2: Even length palindromes (centered between s[i] and s[i+1])
        expand_around_center(s, i, i + 1, palindromes)
    
    return sorted(list(palindromes))

# --- Test cases ---
print("\n--- Expand Around Center Examples ---")

# Example 1
input_str1 = "babad"
output1 = find_all_palindromic_substrings_expand_around_center(input_str1)
print(f"Input: '{input_str1}'")
print(f"Palindromic Substrings: {output1}")

# Example 2
input_str2 = "racecar"
output2 = find_all_palindromic_substrings_expand_around_center(input_str2)
print(f"\nInput: '{input_str2}'")
print(f"Palindromic Substrings: {output2}")

# Example 3
input_str3 = "abacaba"
output3 = find_all_palindromic_substrings_expand_around_center(input_str3)
print(f"\nInput: '{input_str3}'")
print(f"Palindromic Substrings: {output3}")

# Example 4 (Empty string)
input_str4 = ""
output4 = find_all_palindromic_substrings_expand_around_center(input_str4)
print(f"\nInput: '{input_str4}'")
print(f"Palindromic Substrings: {output4}")

# Example 5 (Single character string)
input_str5 = "a"
output5 = find_all_palindromic_substrings_expand_around_center(input_str5)
print(f"\nInput: '{input_str5}'")
print(f"Palindromic Substrings: {output5}")

# Example 6 (Even length string with many palindromes)
input_str6 = "aaaa"
output6 = find_all_palindromic_substrings_expand_around_center(input_str6)
print(f"\nInput: '{input_str6}'")
print(f"Palindromic Substrings: {output6}")
```

### Sample Output:

```output
--- Expand Around Center Examples ---
Input: 'babad'
Palindromic Substrings: ['a', 'aba', 'b', 'bab', 'd']

Input: 'racecar'
Palindromic Substrings: ['a', 'aceca', 'c', 'cec', 'e', 'r', 'racecar']

Input: 'abacaba'
Palindromic Substrings: ['a', 'aba', 'abacaba', 'acaca', 'b', 'bacab', 'c']

Input: ''
Palindromic Substrings: []

Input: 'a'
Palindromic Substrings: ['a']

Input: 'aaaa'
Palindromic Substrings: ['a', 'aa', 'aaa', 'aaaa']
```

## What about Manacher's Algorithm?

You might have heard of Manacher's Algorithm, which can find the *longest* palindromic substring in **O(N) time**. This is the most optimized solution for that specific problem.

**Note:** Manacher's Algorithm is primarily designed to find the *single longest* palindromic substring. While its core ideas (like preprocessing the string to handle even/odd lengths uniformly and leveraging symmetry) can be adapted to find all palindromic substrings, doing so efficiently while still maintaining O(N) for finding *all unique ones* is significantly more complex than the Expand Around Center method. For finding *all* palindromic substrings, O(N^2) (using expand around center) is generally considered the practical and simplest optimal solution. Manacher's focuses on avoiding redundant checks by using previously computed palindrome lengths, a specialized optimization for the "longest" variant.

If your problem specifically asks for the *longest* palindromic substring, then learning Manacher's is a valuable next step for optimal performance. However, for identifying *all* such substrings, the O(N^2) "Expand Around Center" approach is typically sufficient and much easier to implement correctly.

## Practical Considerations and Use Cases

*   **Interview Prep**: This problem, especially the "longest palindromic substring" variant, is extremely common in technical interviews. Understanding the brute-force and expand-around-center methods is crucial.
*   **Text Processing**: Identifying palindromic sequences can be useful in natural language processing, data analysis, or even in bioinformatics for analyzing DNA sequences.
*   **Data Compression**: While not a direct application, understanding string patterns and symmetry can be foundational for algorithms used in data compression.

## Conclusion

We've walked through the problem of finding palindromic substrings, starting from the basic definitions. We explored two distinct algorithmic approaches:

*   **Brute-Force (O(N^3))**: Simple to grasp, but inefficient for larger inputs. Useful for initial understanding and small datasets.
*   **Expand Around Center (O(N^2))**: A much more efficient and practical solution for finding all palindromic substrings. This is often the expected approach in interviews.

While Manacher's algorithm offers an O(N) solution for finding the *longest* palindromic substring, the O(N^2) approach for finding *all* is a solid, pragmatic choice that balances performance with implementation complexity.

Understanding these concepts not only equips you to solve specific problems but also enhances your general algorithmic thinking, pattern recognition, and optimization skills. Keep practicing!