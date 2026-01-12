---
---

# Non-Tail Recursion

## Abstract
**Non-Tail Recursion** is the standard form of recursion where the recursive call is **not** the last instruction in the function. The function performs some work **after** the recursive call returns (e.g., adding a value to the result). This requires the runtime to maintain the current stack frame until the recursive step completes.

## Development

### Core Concept
The "work" is done during the **unwinding** phase (popping the stack).
Example: `return n * factorial(n-1)`. The multiplication by `n` happens *after* `factorial(n-1)` returns.

### Comparison
| Feature | Tail Recursion | Non-Tail Recursion |
|---------|----------------|--------------------|
| **Last Action** | Recursive Call | Calculation / Logic |
| **Stack Usage** | Can be $O(1)$ (with TCO) | Always $O(n)$ |
| **Order** | Work usually done on way down | Work usually done on way up |

### Complexity
- **Time**: $O(n)$
- **Space**: $O(n)$ (Linear stack growth).

## Code Examples (Go)

### Standard Factorial (Non-Tail)
```go
package main

import "fmt"

func Factorial(n int) int {
    if n <= 1 {
        return 1
    }
    // NOT tail-recursive. 
    // Must wait for Factorial(n-1) to return before multiplying by n.
    return n * Factorial(n-1) 
}

func main() {
    fmt.Println(Factorial(5))
}
```

### Tree Traversal (Non-Tail)
Most tree traversals (DFS) are naturally non-tail recursive because they make multiple recursive calls or process data after a call.

```go
func DFS(node *TreeNode) {
    if node == nil {
        return
    }
    DFS(node.Left)  // Non-tail
    fmt.Println(node.Val)
    DFS(node.Right) // Tail position, but previous call makes function generally non-tail
}
```

## Go Application
- **Divide and Conquer**: Algorithms like Merge Sort are inherently non-tail recursive (split, recurse left, recurse right, then merge).
- **Tree/Graph Algorithms**: The most common use case for recursion in Go.

## Interview Preparation
1.  **Risk**: What is the danger of non-tail recursion?
    -   *Answer*: Stack Overflow. If recursion depth > stack limit.
2.  **Conversion**: Can any non-tail recursion be converted to tail recursion?
    -   *Answer*: Yes, usually by introducing an accumulator or using Continuation-Passing Style (CPS), though often just converting to an explicit Stack (iteration) is easier.
