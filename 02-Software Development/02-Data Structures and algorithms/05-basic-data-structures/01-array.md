---
---

# Arrays

## Summary
An **Array** in Go is a fixed-size sequence of elements of the same type. It is a **Value Type**, meaning assigning an array to a new variable creates a full copy of its contents. While rarely used directly in favor of Slices, arrays provide the backing storage for slices and offer zero-allocation performance for small, fixed collections.

## Detailed Explanation

### 1. Core Characteristics
*   **Fixed Size**: The size is part of the type. `[5]int` and `[10]int` are completely distinct types and cannot be compared.
*   **Contiguous Memory**: Elements are stored side-by-side in memory, ensuring $O(1)$ access and perfect cache locality.
*   **Value Semantics**: Passing an array to a function copies the *entire* array (not just a pointer).

### 2. Array vs. Slice
| Feature | Array `[N]T` | Slice `[]T` |
| :--- | :--- | :--- |
| **Size** | Fixed at compile time | Dynamic (growable) |
| **Assignment** | Copies all data | Copies header (ptr, len, cap) |
| **Usage** | Low-level optimization | General purpose standard |
| **Memory** | Stack (often) | Heap (usually) |

## Code Examples (Go)

### 1. Declaration and Initialization
```go
package main

import "fmt"

func main() {
    // Standard declaration (Zero-valued: [0 0 0])
    var a [3]int
    
    // Array Literal
    b := [3]int{10, 20, 30}
    
    // Ellipsis (Compiler counts elements)
    c := [...]string{"Go", "Rust", "C++"} // Type is [3]string
    
    fmt.Printf("Type of c: %T\n", c)
}
```

### 2. Value Semantics (The Trap)
```go
func modify(arr [3]int) {
    arr[0] = 999 // Modifies the COPY, not the original
}

func main() {
    original := [3]int{1, 2, 3}
    modify(original)
    // original is still [1, 2, 3]
}
```
*To modify an array in a function, you must pass a pointer `*[3]int`, or more commonly, pass a slice.*

### 3. Iteration
```go
arr := [3]int{10, 20, 30}

// Range loop (Standard)
for i, v := range arr {
    fmt.Printf("Index: %d, Value: %d\n", i, v)
}
```

## Go Application
*   **Backing Store**: Every slice is backed by an invisible array.
*   **UUIDs/Hashes**: Fixed-length cryptographic hashes (like SHA-256) are returned as `[32]byte` arrays.
*   **Matrix Math**: Fixed-size matrices (e.g., 4x4 transformation matrices in graphics) use arrays to avoid GC overhead.

## Interview Questions

**Q: What is the difference between `[3]int` and `[]int` in Go?**
**A:** `[3]int` is an **Array** (fixed size, value type). `[]int` is a **Slice** (dynamic view over an array, reference-like).

**Q: Why would you use an array instead of a slice?**
**A:** For performance in hot paths. If you need a small, fixed sequence (like a 3D vector `[3]float64`), an array can be allocated entirely on the **Stack**, avoiding Heap allocation and GC overhead.

**Q: If you pass a large array (e.g., `[1MB]byte`) to a function, what happens?**
**A:** Go copies the entire 1MB of memory onto the function's stack. This is slow and risks a Stack Overflow. Always use a slice or a pointer for large datasets.
