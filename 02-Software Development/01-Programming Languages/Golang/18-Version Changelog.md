# Version Changelog

## Summary
A chronological log of major changes in the Go language and toolchain, focusing on features that impact daily development (Generics, Workspaces, Iterators) and critical performance improvements (PGO, Memory Model).

## Detailed Explanation

### Go 1.18 (March 2022) - The Generics Era
*   **Generics**: Added type parameters (`[T any]`) to functions and types. Introduction of the `any` keyword (alias for `interface{}`).
*   **Fuzzing**: Built-in support for fuzz testing (`testing.F`) to find edge cases automatically.
*   **Workspaces**: Multi-module development with `go.work` files, allowing seamless local replacements of dependencies.

### Go 1.19 (August 2022)
*   **Memory Model**: Formalized memory model for atomic operations, aligning with C++20. Added `atomic.Pointer[T]`.
*   **Doc Comments**: Support for links, lists, and headings in doc comments.
*   **Soft Memory Limit**: `GOMEMLIMIT` environment variable to better tune GC in containerized environments.

### Go 1.20 (February 2023)
*   **Profile-Guided Optimization (PGO)**: Preview support. The compiler can use a CPU profile (`default.pgo`) to optimize hot paths (inline functions, devirtualize calls).
*   **Multierrors**: `errors.Join` supports wrapping multiple errors.
*   **Comparability**: `comparable` constraint is now satisfied by interface types even if the underlying type isn't comparable (runtime panic instead of compile error).

### Go 1.21 (August 2023)
*   **Built-ins**: Added `min`, `max`, and `clear` (for maps/slices).
*   **Standard Library**:
    *   `log/slog`: Structured logging.
    *   `slices` and `maps`: Generic utility functions (`slices.Sort`, `maps.Clone`).
    *   `cmp`: Comparability utilities.
*   **PGO**: Now General Availability (GA).
*   **Loop Variable Preview**: `GOEXPERIMENT=loopvar` introduced to fix loop variable capture semantics.

### Go 1.22 (February 2024)
*   **Loop Variable Fix**: Loop variables are now scoped per iteration, not per loop. No more `v := v` inside loops!
*   **Range Over Integers**: `for i := range 10` is now valid.
*   **math/rand/v2**: Faster, safer, and cleaner random number generation.
*   **HTTP Routing**: `net/http.ServeMux` now supports methods (`GET /path`) and wildcards (`/items/{id}`).

### Go 1.23 (August 2024)
*   **Iterators**: Standardized iterators via `iter` package (`Seq`, `Seq2`).
*   **Range Over Func**: The `range` clause now supports iterator functions (`func(func() bool)`).
*   **Unique**: New `unique` package for canonicalizing comparable values (interning strings/structs).

### Go 1.24 (Expected Feb 2025)
*   **Weak Pointers**: `weak` package for weak references (GC can collect the object).
*   **Map Implementation**: Likely switch to Swiss Tables for better performance.
*   **Tool dependencies**: Better management of tool dependencies in `go.mod`.

### Code Example: Evolution of Loops and Generics

```go
package main

import (
    "fmt"
    "slices"
)

// Go 1.18+: Generics
func Reverse[T any](s []T) {
    for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
        s[i], s[j] = s[j], s[i]
    }
}

func main() {
    nums := []int{1, 2, 3}
    Reverse(nums)
    
    // Go 1.21+: slices package
    slices.Sort(nums)
    
    // Go 1.22+: Loop variable scoping (safe to capture pointers)
    var ptrs []*int
    for _, v := range nums {
        // Pre-1.22: This would capture the same address
        // 1.22+: Captures the value of the current iteration
        ptrs = append(ptrs, &v) 
    }
    
    // Go 1.23+: Range over integers
    for i := range 5 {
        fmt.Println(i) // 0, 1, 2, 3, 4
    }
}
```

## Interview Questions

**Q: What changed with loop variables in Go 1.22?**
**A:** Before 1.22, loop variables were reused across iterations, causing bugs when capturing their address or using them in closures (goroutines). In 1.22, the variable is effectively redeclared for each iteration, fixing this common "gotcha".

**Q: What is Profile-Guided Optimization (PGO) in Go?**
**A:** PGO allows the compiler to use a runtime profile (CPU pprof) from a previous run to make optimization decisions, such as which functions to inline. It typically yields a 2-10% performance improvement.

**Q: What is the purpose of the `any` keyword introduced in Go 1.18?**
**A:** `any` is a type alias for `interface{}`. It was introduced alongside Generics to make function signatures with type parameters (like `[T any]`) more readable.
