---
---

# Stacks

## Summary
A **Stack** is a linear data structure that follows the **LIFO** (Last-In, First-Out) principle. The last element added is the first one removed. In Go, stacks are almost exclusively implemented using **Slices**, utilizing `append()` for pushing and slice indexing for popping.

## Detailed Explanation

### 1. Operations
*   **Push**: Add an element to the top. ($O(1)$ amortized)
*   **Pop**: Remove and return the top element. ($O(1)$)
*   **Peek/Top**: View the top element without removing it. ($O(1)$)
*   **IsEmpty**: Check if the stack has no elements.

### 2. Use Cases
*   **Function Call Stack**: Tracking recursive calls.
*   **Undo Mechanisms**: Storing history of actions.
*   **Parsing**: Syntax analysis (checking balanced parentheses).
*   **DFS**: Depth-First Search in graph algorithms.

## Code Examples (Go)

### Idiomatic Slice Implementation
Go doesn't need a special `Stack` class. Slices do the job perfectly.

```go
package main

import "fmt"

func main() {
    // 1. Initialize
    var stack []string

    // 2. Push (Append)
    stack = append(stack, "Page 1")
    stack = append(stack, "Page 2")
    stack = append(stack, "Page 3")

    // 3. Peek
    if len(stack) > 0 {
        top := stack[len(stack)-1]
        fmt.Println("Current Top:", top) // "Page 3"
    }

    // 4. Pop
    if len(stack) > 0 {
        // Get value
        index := len(stack) - 1
        poppedValue := stack[index]
        
        // Remove from stack (Slicing)
        // Note: For pointers, set stack[index] = nil to prevent memory leaks
        stack = stack[:index]
        
        fmt.Println("Popped:", poppedValue)
    }
}
```

### Generic Stack (Struct Wrapper)
For stricter type safety or clearer intent, you can wrap a slice in a generic struct.

```go
type Stack[T any] struct {
    data []T
}

func (s *Stack[T]) Push(v T) {
    s.data = append(s.data, v)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.data) == 0 {
        var zero T
        return zero, false
    }
    index := len(s.data) - 1
    val := s.data[index]
    s.data = s.data[:index]
    return val, true
}
```

## Interview Questions

**Q: How do you implement a Stack using Slices in Go?**
**A:** Use `append(slice, val)` to Push. Use `slice[len(slice)-1]` to Peek. Use `slice = slice[:len(slice)-1]` to Pop.

**Q: What is the "Memory Leak" pitfall when Popping pointers from a Stack in Go?**
**A:** If the stack holds pointers (e.g., `[]*User`), simply reducing the slice length (`stack[:n-1]`) leaves the underlying array referencing the object. The Garbage Collector cannot free that object. You must explicitly set `stack[n-1] = nil` before slicing.

**Q: How would you design a Stack that supports `Min()` (getting the minimum element) in $O(1)$?**
**A:** maintain a second stack (a "min-stack") that tracks the minimums. When pushing `x` to the main stack, push `min(x, currentMin)` to the min-stack. When popping from the main stack, pop from the min-stack too.
