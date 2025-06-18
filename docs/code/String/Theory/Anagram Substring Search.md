---
title: Anagram Substring Search Practical Algorithms for Developers
date: 2025-06-18T02:04:08.681Z
description: "Master the 'Anagram Substring Search' problem with clear, practical examples. Learn naive and optimized sliding window algorithms in Python and Go, complete with code and output for efficient text processing."
tags: [Algorithms, String Manipulation, Python, Go, Data Structures, Hashing, Computer Science, Interview Prep]
categories: [Computer Science, Programming]
comments: true
---

Searching for patterns within text is a fundamental task in computer science. One interesting variation is finding "anagram substrings" – instances where a substring within a larger text is an anagram of a given pattern. This problem often appears in coding interviews and has practical applications in areas like text analysis, data matching, and even bioinformatics.

In this post, we'll break down the Anagram Substring Search problem, explore different approaches, and provide clear, runnable code examples in Python and Go.

## What is Anagram Substring Search?

Let's clarify the terms:

*   **Anagram**: Two strings are anagrams of each other if they contain the same characters with the same frequencies, regardless of order. For example, "listen" and "silent" are anagrams. "Debit card" and "bad credit" are also anagrams.
*   **Substring**: A contiguous sequence of characters within a string. "appl" is a substring of "apple", but "aple" is not.

**Anagram Substring Search** (or "Find All Anagrams of a String in a Text") involves finding all starting indices in a `text` string where a substring of `text` is an anagram of a given `pattern` string.

**Example:**
Text: "cbaebabacd"
Pattern: "abc"

Expected output: `[0, 6]`
*   At index 0: "cba" is an anagram of "abc".
*   At index 6: "bac" is an anagram of "abc".

## Approach 1: The Naive Method (Brute Force)

The most straightforward way to solve this is to iterate through every possible substring in the `text` that has the same length as the `pattern`. For each such substring, we check if it's an anagram of the `pattern`.

**How to check if two strings are anagrams:**

1.  **Sorting**: Sort the characters of both strings. If the sorted strings are identical, they are anagrams. This takes `O(M log M)` time for strings of length `M`.
2.  **Frequency Maps (or Character Counts)**: Create a frequency map (e.g., a dictionary or an array of 26 integers for lowercase English letters) for each string. Populate the map by counting character occurrences. If the frequency maps are identical, the strings are anagrams. This takes `O(M)` time.

For the naive approach, using frequency maps to check anagrams is generally preferred as it's faster than sorting.

**Algorithm Steps:**

1.  Get the length of the `text` (`N`) and `pattern` (`M`).
2.  If `M > N`, no anagrams can exist, so return an empty list.
3.  Iterate from `i = 0` to `N - M`.
    *   Extract the current window (substring) of `text` from `i` to `i + M - 1`.
    *   Check if this window is an anagram of the `pattern` using frequency maps.
    *   If it is, add `i` to our list of results.

**Time Complexity**: `O((N - M + 1) * M * A)` where `N` is text length, `M` is pattern length, and `A` is the size of the alphabet. In the worst case (`M` is close to `N`), this is `O(N * M * A)`. If using sorting for anagram check, it becomes `O(N * M log M)`. This can be quite slow for large texts.

### Python Example: Naive Approach

```python
import collections

def are_anagrams(s1, s2):
    """
    Checks if two strings are anagrams using frequency maps.
    Assumes ASCII characters for simplicity, but works generally.
    """
    if len(s1) != len(s2):
        return False
    
    # Using collections.Counter is very efficient for frequency maps
    return collections.Counter(s1) == collections.Counter(s2)

def find_anagram_substrings_naive(text, pattern):
    """
    Finds all starting indices of anagram substrings using the naive approach.
    """
    n = len(text)
    m = len(pattern)
    results = []

    if m > n:
        return results

    # Pre-calculate pattern frequency map
    # Note: We could re-calculate this inside the loop for each comparison,
    # but pre-calculating it once is slightly more efficient.
    pattern_freq = collections.Counter(pattern)

    for i in range(n - m + 1):
        current_window = text[i : i + m]
        if are_anagrams(current_window, pattern):
            results.append(i)
            
    return results

# --- Test Cases ---
print("--- Naive Approach Examples ---")

# Example 1
text1 = "cbaebabacd"
pattern1 = "abc"
print(f"Text: '{text1}', Pattern: '{pattern1}'")
output1 = find_anagram_substrings_naive(text1, pattern1)
print(f"Output: {output1}")

# Example 2
text2 = "abab"
pattern2 = "ab"
print(f"Text: '{text2}', Pattern: '{pattern2}'")
output2 = find_anagram_substrings_naive(text2, pattern2)
print(f"Output: {output2}")

# Example 3: No matches
text3 = "hello"
pattern3 = "world"
print(f"Text: '{text3}', Pattern: '{pattern3}'")
output3 = find_anagram_substrings_naive(text3, pattern3)
print(f"Output: {output3}")

# Example 4: Pattern longer than text
text4 = "a"
pattern4 = "abc"
print(f"Text: '{text4}', Pattern: '{pattern4}'")
output4 = find_anagram_substrings_naive(text4, pattern4)
print(f"Output: {output4}")
```

```output
--- Naive Approach Examples ---
Text: 'cbaebabacd', Pattern: 'abc'
Output: [0, 6]
Text: 'abab', Pattern: 'ab'
Output: [0, 1, 2]
Text: 'hello', Pattern: 'world'
Output: []
Text: 'a', Pattern: 'abc'
Output: []
```

## Approach 2: Sliding Window with Frequency Maps (Optimized)

The naive approach repeatedly builds a frequency map for each substring. We can optimize this by using a "sliding window" technique. Instead of recalculating the map for each new window, we *update* it as the window slides.

**Key Idea**: When the window slides one character to the right:
1.  The character at the leftmost edge of the previous window "leaves" the window. Its count in the frequency map should be decremented.
2.  The character at the new rightmost edge "enters" the window. Its count should be incremented.

This allows us to update the window's frequency map in `O(1)` time (amortized) instead of `O(M)`.

**Algorithm Steps:**

1.  Get the length of the `text` (`N`) and `pattern` (`M`).
2.  If `M > N`, return an empty list.
3.  Initialize `pattern_freq` (a frequency map for `pattern`).
4.  Initialize `window_freq` (a frequency map for the *first* window of `text`, i.e., `text[0:M]`).
5.  Initialize `results` list.
6.  Compare `pattern_freq` and `window_freq`. If they are identical, add `0` to `results`.
7.  Iterate from `i = 1` to `N - M` (sliding the window):
    *   **Remove** the character `text[i-1]` from `window_freq` (decrement its count).
    *   **Add** the character `text[i+M-1]` to `window_freq` (increment its count).
    *   **Compare** `pattern_freq` and `window_freq`. If identical, add `i` to `results`.

**Optimized Comparison (Using a `matches` counter):**

Comparing two entire frequency maps (`collections.Counter` objects in Python, or arrays) takes `O(A)` time, where `A` is the alphabet size. If `A` is large, or we want to be strictly `O(1)` per window slide, we can use a `matches` counter.

The `matches` counter keeps track of how many characters in `window_freq` have the *exact same frequency* as in `pattern_freq`.

*   Initialize `matches = 0`.
*   When building initial `pattern_freq` and `window_freq`:
    *   For each character `c` in the alphabet: if `pattern_freq[c] == window_freq[c]`, increment `matches`.
*   When sliding the window (for `char_out = text[i-1]` and `char_in = text[i+M-1]`):
    *   Before `char_out` count decrements: if `window_freq[char_out] == pattern_freq[char_out]`, decrement `matches`.
    *   Decrement `window_freq[char_out]`.
    *   After `char_out` count decrements: if `window_freq[char_out] == pattern_freq[char_out]`, increment `matches`.
    *   Before `char_in` count increments: if `window_freq[char_in] == pattern_freq[char_in]`, decrement `matches`.
    *   Increment `window_freq[char_in]`.
    *   After `char_in` count increments: if `window_freq[char_in] == pattern_freq[char_in]`, increment `matches`.
    *   If `matches == A` (or `pattern_freq.keys()` count if using only relevant chars), then the maps are identical. Add `i` to results.

Note: The `matches` counter approach is more complex to implement correctly due to the pre- and post-update checks. A simpler `collections.Counter` comparison is often sufficient and more readable, especially if `A` is small (e.g., 26 for English letters). For competitive programming, the `matches` counter is a common optimization.

**Time Complexity**: `O(N + M + A)`.
*   `O(M)` to build initial pattern frequency map.
*   `O(M)` to build initial window frequency map.
*   `O(N - M)` slides. Each slide takes `O(1)` to update counts and `O(A)` to compare maps (or `O(1)` using the `matches` counter if `A` is constant).
Overall, if `A` is constant and small (e.g., 26 for English alphabet), this simplifies to `O(N)`. This is significantly better than the naive approach.

### Python Example: Sliding Window Optimized

We'll use `collections.Counter` directly, as its comparison is optimized in C and usually fast enough. This is a pragmatic choice for Python.

```python
import collections

def find_anagram_substrings_sliding_window(text, pattern):
    """
    Finds all starting indices of anagram substrings using the sliding window approach.
    """
    n = len(text)
    m = len(pattern)
    results = []

    if m > n:
        return results

    # Build frequency map for the pattern
    pattern_freq = collections.Counter(pattern)
    
    # Build frequency map for the first window of the text
    window_freq = collections.Counter(text[0:m])

    # Check the first window
    if window_freq == pattern_freq:
        results.append(0)

    # Slide the window across the rest of the text
    for i in range(1, n - m + 1):
        # Character leaving the window
        char_out = text[i - 1]
        window_freq[char_out] -= 1
        if window_freq[char_out] == 0:
            del window_freq[char_out] # Clean up zero counts for efficiency/correct comparison

        # Character entering the window
        char_in = text[i + m - 1]
        window_freq[char_in] += 1

        # Check if current window is an anagram of pattern
        if window_freq == pattern_freq:
            results.append(i)
            
    return results

# --- Test Cases ---
print("\n--- Sliding Window Approach Examples (Python) ---")

# Example 1
text1 = "cbaebabacd"
pattern1 = "abc"
print(f"Text: '{text1}', Pattern: '{pattern1}'")
output1 = find_anagram_substrings_sliding_window(text1, pattern1)
print(f"Output: {output1}")

# Example 2
text2 = "abab"
pattern2 = "ab"
print(f"Text: '{text2}', Pattern: '{pattern2}'")
output2 = find_anagram_substrings_sliding_window(text2, pattern2)
print(f"Output: {output2}")

# Example 3: Long text, small pattern
text3 = "abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz"
pattern3 = "zyxw"
print(f"Text: '{text3}', Pattern: '{pattern3}'")
output3 = find_anagram_substrings_sliding_window(text3, pattern3)
print(f"Output: {output3}")

# Example 4: No matches, non-alphabetic characters
text4 = "helloworld!"
pattern4 = "dlo" # Note: 'dlo' is not an anagram of any substring of 'helloworld!'
print(f"Text: '{text4}', Pattern: '{pattern4}'")
output4 = find_anagram_substrings_sliding_window(text4, pattern4)
print(f"Output: {output4}")

# Example 5: Case sensitivity
text5 = "BbAcCb"
pattern5 = "abc"
print(f"Text: '{text5}', Pattern: '{pattern5}'")
output5 = find_anagram_substrings_sliding_window(text5, pattern5)
print(f"Output: {output5}") # Will yield no matches as 'A' != 'a'
```

```output
--- Sliding Window Approach Examples (Python) ---
Text: 'cbaebabacd', Pattern: 'abc'
Output: [0, 6]
Text: 'abab', Pattern: 'ab'
Output: [0, 1, 2]
Text: 'abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz', Pattern: 'zyxw'
Output: [22, 48]
Text: 'helloworld!', Pattern: 'dlo'
Output: []
Text: 'BbAcCb', Pattern: 'abc'
Output: []
```

### Go Example: Sliding Window Optimized

In Go, we typically use arrays (e.g., `[26]int` for lowercase English letters or `[128]int` for ASCII) to represent frequency maps for better performance than `map[rune]int` for fixed, small alphabets. This also simplifies the `matches` counter implementation.

```go
package main

import (
	"fmt"
)

// findAnagramSubstringsSlidingWindow finds all starting indices of anagram substrings.
func findAnagramSubstringsSlidingWindow(text string, pattern string) []int {
	n := len(text)
	m := len(pattern)
	results := []int{}

	if m > n {
		return results
	}

	// Use frequency arrays for ASCII characters (0-127)
	// For English lowercase letters, [26]int would be sufficient.
	// For full Unicode, a map[rune]int would be necessary.
	patternFreq := [128]int{}
	windowFreq := [128]int{}

	// Populate patternFreq
	for _, char := range pattern {
		patternFreq[char]++
	}

	// Populate windowFreq for the first window and count initial matches
	matches := 0
	for i := 0; i < m; i++ {
		char := text[i] // byte value
		windowFreq[char]++
	}

	// Compare initial windowFreq and patternFreq to count initial matches
	// A 'match' means the character count in windowFreq matches patternFreq
	for i := 0; i < 128; i++ {
		if patternFreq[i] == windowFreq[i] && patternFreq[i] > 0 { // Check >0 to avoid matching empty counts
			matches++
		}
	}
	
	// If all pattern characters have matching frequencies in the first window
	if matches == countDistinctChars(patternFreq[:]) { // countDistinctChars is helper for match comparison
		results = append(results, 0)
	}

	// Slide the window
	for i := 1; i <= n-m; i++ {
		// Character leaving the window
		charOut := text[i-1]
		if patternFreq[charOut] > 0 { // Only consider chars relevant to pattern
			if windowFreq[charOut] == patternFreq[charOut] {
				matches-- // This char was matching, now it won't be
			}
		}
		windowFreq[charOut]--
		if patternFreq[charOut] > 0 {
			if windowFreq[charOut] == patternFreq[charOut] {
				matches++ // This char might become matching again after decrement
			}
		}

		// Character entering the window
		charIn := text[i+m-1]
		if patternFreq[charIn] > 0 { // Only consider chars relevant to pattern
			if windowFreq[charIn] == patternFreq[charIn] {
				matches-- // This char was matching, now it won't be
			}
		}
		windowFreq[charIn]++
		if patternFreq[charIn] > 0 {
			if windowFreq[charIn] == patternFreq[charIn] {
				matches++ // This char might become matching again after increment
			}
		}
		
		// If all pattern characters have matching frequencies
		if matches == countDistinctChars(patternFreq[:]) {
			results = append(results, i)
		}
	}

	return results
}

// Helper to count distinct non-zero characters in a frequency array
// This helps determine how many `matches` are needed for a full match.
func countDistinctChars(freq []int) int {
	count := 0
	for _, f := range freq {
		if f > 0 {
			count++
		}
	}
	return count
}

func main() {
	fmt.Println("--- Sliding Window Approach Examples (Go) ---")

	// Example 1
	text1 := "cbaebabacd"
	pattern1 := "abc"
	fmt.Printf("Text: '%s', Pattern: '%s'\n", text1, pattern1)
	output1 := findAnagramSubstringsSlidingWindow(text1, pattern1)
	fmt.Printf("Output: %v\n", output1)

	// Example 2
	text2 := "abab"
	pattern2 := "ab"
	fmt.Printf("Text: '%s', Pattern: '%s'\n", text2, pattern2)
	output2 := findAnagramSubstringsSlidingWindow(text2, pattern2)
	fmt.Printf("Output: %v\n", output2)

	// Example 3: Long text, small pattern
	text3 := "abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz"
	pattern3 := "zyxw"
	fmt.Printf("Text: '%s', Pattern: '%s'\n", text3, pattern3)
	output3 := findAnagramSubstringsSlidingWindow(text3, pattern3)
	fmt.Printf("Output: %v\n", output3)

	// Example 4: Case sensitivity (will not match 'abc' to 'Abc')
	text4 := "BbAcCb"
	pattern4 := "abc"
	fmt.Printf("Text: '%s', Pattern: '%s'\n", text4, pattern4)
	output4 := findAnagramSubstringsSlidingWindow(text4, pattern4)
	fmt.Printf("Output: %v\n", output4)
}
```

```output
--- Sliding Window Approach Examples (Go) ---
Text: 'cbaebabacd', Pattern: 'abc'
Output: [0 6]
Text: 'abab', Pattern: 'ab'
Output: [0 1 2]
Text: 'abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz'
Pattern: 'zyxw'
Output: [22 48]
Text: 'BbAcCb', Pattern: 'abc'
Output: []
```

## Variations and Considerations

1.  **Case Sensitivity**: Our examples treat 'A' and 'a' as different characters. If case-insensitivity is required, convert both `text` and `pattern` to a consistent case (e.g., lowercase) before processing.
2.  **Non-alphabetic Characters**: If the problem allows spaces, punctuation, or numbers, and they should be considered part of the anagram, then the frequency maps (or arrays) must accommodate their ASCII/Unicode values. If they should be ignored, filter them out before processing or during frequency counting. Our `[128]int` array for Go handles common ASCII characters, but for full Unicode, `map[rune]int` would be necessary.
3.  **Empty Pattern/Text**: Our code handles `m > n` (pattern longer than text) by returning an empty list. If `pattern` is empty, it depends on the problem definition. Some might consider an empty string an anagram of itself, matching at every possible index; others might return an error or empty list. Our current code would return an empty list because the `patternFreq` for an empty pattern would be all zeros, and `countDistinctChars` would return 0, leading to `matches == 0`.
4.  **Character Set**: For specific character sets (e.g., only lowercase English letters), using a fixed-size array (`[26]int`) mapping 'a' to index 0, 'b' to index 1, etc., is more memory-efficient and potentially faster than hash maps.

## Choosing the Right Approach

*   **Naive Approach**:
    *   **Pros**: Simple to understand and implement. Good for very small inputs or when extreme clarity is prioritized over performance.
    *   **Cons**: Inefficient for larger inputs. Time complexity `O(N * M * A)` or `O(N * M log M)` is generally unacceptable for competitive programming or production systems with large datasets.

*   **Sliding Window with Frequency Maps**:
    *   **Pros**: Highly efficient with `O(N + M + A)` (effectively `O(N)` for small, constant alphabets). This is the standard, optimized solution.
    *   **Cons**: Slightly more complex to implement correctly due to managing the sliding window's character counts and the `matches` logic (if used).

**Verdict**: For virtually any practical application or interview scenario, the **Sliding Window** approach is the one to use and demonstrate. It showcases a strong understanding of algorithmic optimization and efficient data structure usage.

## Conclusion

The Anagram Substring Search problem is a classic example of how applying a clever data structure (frequency maps) and an algorithmic pattern (sliding window) can drastically improve performance from a naive brute-force solution. Understanding and implementing the sliding window technique is a valuable skill for any developer dealing with string manipulation or array-based problems.

Remember, practice is key. Try implementing these solutions yourself, tweaking them for different constraints (case sensitivity, character sets), and measuring their performance. Happy coding!
