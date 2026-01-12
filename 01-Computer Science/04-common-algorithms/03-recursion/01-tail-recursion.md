---
---

# Tail Recursion

## Abstract
**Tail Recursion** is a special form of recursion where the recursive call is the **very last action** performed by the function. In languages that support **Tail Call Optimization (TCO)**, the compiler can optimize this into a simple loop (jump), reusing the current stack frame. This prevents stack overflow errors and saves memory.

## Development

### Core Concept
- **Standard Recursion**: The function must wait for the recursive call to return to perform a final calculation (e.g., `return 1 + recurse(n-1)`). This requires holding the stack frame.
- **Tail Recursion**: The function has nothing left to do after the recursive call returns except return the result (e.g., `return recurse(n-1, acc+1)`). The state is passed forward.

### Go and TCO
**Go does NOT support Tail Call Optimization**.
The Go compiler (gc) deliberately avoids TCO to preserve accurate stack traces for debugging and panic recovery. While you can write tail-recursive code in Go, it will still consume stack frames like any other recursion. However, since Go stacks are dynamic and can grow up to 1GB (on 64-bit systems), it is less prone to stack overflows than languages with fixed stacks (like C/Java default).

### Complexity
- **Time**: $O(n)$
- **Space**: 
    -   With TCO: $O(1)$
    -   Without TCO (Go): $O(n)$

## Code Examples (Go)

### Tail Recursive Factorial
We use an **accumulator** (`acc`) to pass the state forward.

```go
package main

import "fmt"

func FactorialTail(n int) int {
    return factHelper(n, 1)
}

// factHelper is tail-recursive: the return is the recursive call itself.
func factHelper(n int, acc int) int {
    if n <= 1 {
        return acc
    }
    return factHelper(n-1, n*acc)
}

func main() {
    fmt.Println(FactorialTail(5)) // 120
}
```

## Go Application
- **Idiomatic Go**: Since Go lacks TCO, the idiomatic preference is to use **loops** (iterative approach) instead of deep recursion when possible, especially for simple linear tasks.
- **State Machines**: Sometimes used in lexers (like `text/template` in stdlib) where functions return the next function state, simulating TCO via a trampoline loop.

## Interview Preparation
1.  **Does Go implement TCO?**
    -   *Answer*: No. It keeps stack frames for debugging visibility.
2.  **How to convert Recursion to Iteration?**
    -   *Answer*: Use a loop and variables to track state (accumulator), effectively manually implementing the optimization.
