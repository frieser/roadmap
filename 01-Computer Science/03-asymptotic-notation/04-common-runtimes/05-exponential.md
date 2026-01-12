---
---

# Exponential Time - O(2^n)

## Abstract
**Exponential Time**, denoted as **O(2ⁿ)** (or $c^n$), represents a runtime that doubles with each addition to the input data set. This is the **"horrible"** zone of complexity. Algorithms with this runtime become computationally infeasible very quickly, often for $n > 40$. It typically appears in recursive algorithms that solve a problem by branching into two or more sub-problems without memoization.

## Development

### Core Concept
- $n=1 \to 2$ steps
- $n=10 \to 1024$ steps
- $n=20 \to \approx 1$ million steps
- $n=30 \to \approx 1$ billion steps
- $n=64 \to$ More steps than atoms in the universe.

### Common Sources
- **Recursive Fibonacci** (Naive).
- **Power Set** (Finding all subsets).
- **Brute-force solutions** to NP-hard problems (Traveling Salesman).

## Code Examples (Go)

### 1. Naive Recursive Fibonacci
The textbook example of O(2ⁿ). Each call branches into two more calls.
```go
package main

// Fib calculates the nth fibonacci number.
// WARNING: Do not run with n > 50
func Fib(n int) int {
    if n <= 1 {
        return n
    }
    // Each level of recursion doubles the number of calls
    return Fib(n-1) + Fib(n-2)
}
```

### 2. Generating All Subsets (Power Set)
To generate all subsets of a set of size $n$, we have $2^n$ subsets.
```go
func subsets(nums []int) [][]int {
    result := [][]int{}
    var backtrack func(index int, current []int)
    
    backtrack = func(index int, current []int) {
        // Base case: processed all elements
        if index == len(nums) {
            // Must copy the slice
            temp := make([]int, len(current))
            copy(temp, current)
            result = append(result, temp)
            return
        }
        
        // Choice 1: Exclude nums[index]
        backtrack(index+1, current)
        
        // Choice 2: Include nums[index]
        current = append(current, nums[index])
        backtrack(index+1, current)
        // Backtrack (remove last element)
        current = current[:len(current)-1]
    }
    
    backtrack(0, []int{})
    return result
}
```

## Go Application & Ecosystem
- **Optimization**: You almost **never** want O(2ⁿ) in production code.
- **Dynamic Programming**: The primary way to fix exponential recursion is **Memoization** (caching results) or **Tabulation** (iterative approach), usually bringing complexity down to O(n) or O(n²).
- **Backtracking**: Used in constraint satisfaction problems (Sudoku solver), but usually with pruning to avoid the full worst-case exponential path.

## Interview Preparation
1.  **How do you fix O(2ⁿ) Fibonacci?**
    -   *Answer*: Use a map to cache results (Memoization) or use a loop (Iterative). Both reduce it to O(n).
2.  **Why is the Power Set O(2ⁿ)?**
    -   *Answer*: For every element, you have two choices: include it or exclude it. $2 \times 2 \times \dots \times 2$ ($n$ times) = $2^n$.
