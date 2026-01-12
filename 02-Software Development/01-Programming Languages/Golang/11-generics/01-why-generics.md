#Golang
---
---

## Summary

Generics (introduced in Go 1.18) solve the problem of code duplication for algorithms that apply to multiple types. Before generics, developers had to either duplicate functions for each type (violating DRY) or use `interface{}`/reflection (sacrificing type safety and performance). Generics allow writing type-safe code that works with different types by abstracting the specific type logic, enabling reusable data structures and algorithms.

## Detailed Explanation

### The Problem: Code Duplication

Without generics, you need separate functions for each type:

```go
package main

import "fmt"

func SumInts(m map[string]int64) int64 {
    var s int64
    for _, v := range m {
        s += v
    }
    return s
}

func SumFloats(m map[string]float64) float64 {
    var s float64
    for _, v := range m {
        s += v
    }
    return s
}

func main() {
    ints := map[string]int64{"first": 34, "second": 12}
    floats := map[string]float64{"first": 35.98, "second": 26.99}

    fmt.Printf("Generic Sums: %v and %v\n",
        SumInts(ints),
        SumFloats(floats))
}
```

### The Alternative: Interface{} (Unsafe)

Using `interface{}` allows one function, but loses type safety and requires assertions:

```go
func SumAny(nums []interface{}) interface{} {
    // Complex type switching logic...
    // Runtime errors possible!
    // Performance hit due to boxing/unboxing
}
```

### The Solution: Generics

With generics, you write the logic once with type parameters:

```go
package main

import "fmt"

// T is a type parameter restricted to int64 or float64
func Sum[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}

func main() {
    ints := map[string]int64{"first": 34, "second": 12}
    floats := map[string]float64{"first": 35.98, "second": 26.99}

    // Call generic function
    fmt.Printf("Generic Sums: %v and %v\n",
        Sum(ints),
        Sum(floats))
}
```

### Comparison: Approaches to Polymorphism

| Approach | Pros | Cons |
|----------|------|------|
| **Duplication** | Fast, Simple | Violates DRY, hard to maintain |
| **Interfaces** | Flexible, dynamic | Runtime cost, no compile-time checks |
| **Reflection** | Extremely dynamic | Slow, complex, fragile |
| **Generics** | Type-safe, fast (monomorphization) | Slight compile time increase |

### Benefits of Go Generics

1.  **Type Safety**: Errors caught at compile time, not runtime.
2.  **Performance**: No runtime overhead (boxing/assertions). The compiler generates specialized code for each type used.
3.  **Readability**: Expresses intent clearly ("this works for any number") rather than boilerplate.
4.  **Reusability**: Enables standard libraries for slices, maps, and sets (e.g., the `slices` and `maps` packages added in Go 1.21).

### When NOT to Use Generics

Don't use generics if:
*   You are just calling methods on the type arguments (use interfaces instead).
*    The implementation differs significantly for each type.
*   The function signature is complex and hard to read.

> "Write code, don't write types." - Robert Griesemer

### Visualization of Monomorphization

How the compiler handles generics:

```mermaid
flowchart TD
    A[Generic Code<br/>func Min[T](a, b T) T] --> B{Used with int?}
    A --> C{Used with float?}
    
    B -->|Yes| D[Compiler generates:<br/>func Min_int(a, b int) int]
    C -->|Yes| E[Compiler generates:<br/>func Min_float(a, b float) float]
    
    D --> F[Binary Code]
    E --> F
```

## Interview Questions

**Q: Why were generics added to Go after so many years?**

**A:** Go prioritized simplicity and fast compilation. The team waited until they found a design that fit Go's philosophy—efficient compilation, good execution performance (monomorphization), and backward compatibility. Earlier proposals were either too complex (C++ templates) or too slow (Java-style erasure). Go 1.18's design balances these trade-offs using Type Sets.

**Q: What is the difference between using `interface{}` and Generics?**

**A:** `interface{}` accepts any type but discards type information, requiring runtime type assertions and preventing compile-time checks. Generics preserve type information at compile time, ensuring type safety and allowing the compiler to optimize the code (no boxing/unboxing overhead). Generics are strictly better for performance and safety when type identity matters.

**Q: Does Go use type erasure or monomorphization for generics?**

**A:** Go primarily uses a technique similar to monomorphization (stenciling). The compiler generates distinct function instances for different concrete types (e.g., `func[int]` and `func[float64]`). However, for pointer types that have the same underlying memory layout, it may share code (using a "gcshape" approach) to reduce binary size bloat, a common issue with pure C++ templates.

**Q: Can you use generics for methods?**

**A:** You can define methods on a generic type (e.g., `func (s *Stack[T]) Push(v T)`), but you **cannot** introduce new type parameters in a method declaration (e.g., `func (s *Stack[T]) Map[R](f func(T) R)` is invalid). This limitation exists to simplify the implementation and avoid parsing ambiguities.
