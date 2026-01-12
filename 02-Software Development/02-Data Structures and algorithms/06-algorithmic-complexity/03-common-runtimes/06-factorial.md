---
---

# O(n!) Factorial Time

## Summary
**$O(n!)$ Factorial Time** is the fastest growing runtime commonly discussed. It represents algorithms that must generate every possible **permutation** or ordering of the input. It becomes practically impossible to run for $N > 15$.

## Characteristics
*   **Scalability**: Non-existent.
    *   $5! = 120$
    *   $10! = 3.6$ Million
    *   $20! = 2.4 \times 10^{18}$ (Quintillions)
*   **Mechanism**: Permutations, Brute-forcing all paths in a graph.
*   **Graph**: A vertical wall.

## Common Operations
1.  **Permutations**: "Find all ways to rearrange 'ABC'".
2.  **Traveling Salesman Problem (Brute Force)**: Trying every route between cities to find the shortest.
3.  **Solving Sudoku (Naive Backtracking)**.

## Go Code Examples

### Generating Permutations
A simple recursive function to print all orderings.

```go
package main

import "fmt"

// Permute is O(n!)
func Permute(arr []rune, l, r int) {
    if l == r {
        fmt.Println(string(arr))
    } else {
        for i := l; i <= r; i++ {
            // Swap
            arr[l], arr[i] = arr[i], arr[l]
            
            // Recurse
            Permute(arr, l+1, r)
            
            // Backtrack (Swap back)
            arr[l], arr[i] = arr[i], arr[l]
        }
    }
}

func main() {
    str := []rune("ABC")
    n := len(str)
    Permute(str, 0, n-1)
    // Output: ABC, ACB, BAC, BCA, CBA, CAB (3! = 6)
}
```
