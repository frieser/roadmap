---
---

# Factorial Time - O(n!)

## Abstract
**Factorial Time**, denoted as **O(n!)**, is the steepest and worst common time complexity. It grows explosively. $n!$ (n factorial) is the product of all positive integers less than or equal to $n$. It typically arises when generating **all permutations** of a set.

## Development

### The Scale of Explosion
- $n=5 \to 120$ steps
- $n=10 \to 3.6$ million steps
- $n=13 \to 6.2$ billion steps (Limit for reasonable wait time)
- $n=20 \to$ 2 quintillion steps (Centuries)

### Common Sources
- **Permutations**: Generating all possible orderings of a list.
- **Traveling Salesman Problem** (Brute force): Trying every possible route between cities.
- **Heap's Algorithm**.

## Code Examples (Go)

### 1. Generating Permutations
Finding all ways to arrange a slice of numbers.
```go
package main

import "fmt"

func permute(nums []int) [][]int {
    var result [][]int
    var backtrack func(int)

    backtrack = func(first int) {
        // If we have used all positions, we found a permutation
        if first == len(nums) {
            temp := make([]int, len(nums))
            copy(temp, nums)
            result = append(result, temp)
            return
        }

        for i := first; i < len(nums); i++ {
            // Swap current element with the first element of remaining
            nums[first], nums[i] = nums[i], nums[first]
            
            // Recurse for the next position
            backtrack(first + 1)
            
            // Backtrack (swap back)
            nums[first], nums[i] = nums[i], nums[first]
        }
    }

    backtrack(0)
    return result
}

func main() {
    p := permute([]int{1, 2, 3})
    // 3! = 6 results
    fmt.Println(p) 
}
```

## Go Application & Ecosystem
- **Testing**: Sometimes used in "property-based testing" or "fuzzing" to try edge case orderings of events on very small inputs ($n < 10$).
- **Combinatorics Libraries**: Libraries like `gonum` might have permutation helpers, but they are used sparingly.
- **Warning**: Never run O(n!) algorithms on user input without strict length limits (e.g., `if len(input) > 10 { return error }`).

## Interview Preparation
1.  **What is bigger: O(2ⁿ) or O(n!)?**
    -   *Answer*: O(n!) is much bigger. For large n, $n! \gg 2^n$.
2.  **When would you use an O(n!) algorithm?**
    -   *Answer*: Only for very small $n$ (usually $n \le 12$) or when solving combinatorial problems where you typically need to output all possibilities.
3.  **Traveling Salesman Problem**: The brute force is O(n!), but dynamic programming (Held-Karp) can reduce it to $O(n^2 2^n)$, which is still exponential but better.
