#Golang
---
---

## Summary

Generic functions allow you to write a single function that operates on different types while maintaining type safety. They are defined using square brackets `[]` to specify **type parameters** and **constraints**. The syntax `func Name[T Constraint](param T)` declares a generic function. This enables writing algorithms like `Min`, `Max`, `Sort`, or `Map` once, working safely with any compatible type.

## Detailed Explanation

### Syntax

```go
func FunctionName[TypeParam Constraint](params...) returnType {
    // body
}
```

*   **Type Parameters**: Enclosed in `[]` (e.g., `[T any]`).
*   **Constraint**: Defines what operations are allowed on the type (e.g., `any`, `comparable`, `int | float64`).

### Basic Example: Print Anything

```go
package main

import "fmt"

// T is the type parameter. 'any' means T can be any type.
func Print[T any](s []T) {
    for _, v := range s {
        fmt.Print(v, " ")
    }
    fmt.Println()
}

func main() {
    Print([]int{1, 2, 3})       // 1 2 3
    Print([]string{"a", "b"})   // a b
}
```

### Example with Constraints: Min Function

To compare values (e.g., `a < b`), the type must support ordering. `any` is not enough.

```go
package main

import (
    "fmt"
    "golang.org/x/exp/constraints" // Standard constraints package
)

// Ordered constraint allows types supporting <, >, <=, >=
func Min[T constraints.Ordered](a, b T) T {
    if a < b {
        return a
    }
    return b
}

func main() {
    fmt.Println(Min(3, 5))          // 3 (int)
    fmt.Println(Min(1.5, 2.3))      // 1.5 (float64)
    fmt.Println(Min("apple", "banana")) // apple (string)
}
```

### Multiple Type Parameters

Functions can have multiple type parameters.

```go
package main

import "fmt"

// MapKeys returns a slice of keys from a map
// K must be comparable (required for map keys)
// V can be any type
func MapKeys[K comparable, V any](m map[K]V) []K {
    keys := make([]K, 0, len(m))
    for k := range m {
        keys = append(keys, k)
    }
    return keys
}

func main() {
    m := map[string]int{"one": 1, "two": 2}
    keys := MapKeys(m)
    fmt.Println(keys) // [one two] (order random)
}
```

### Constraint "comparable"

The `comparable` constraint is built-in. It allows types that support `==` and `!=`. This is required for map keys.

```go
func Index[T comparable](s []T, x T) int {
    for i, v := range s {
        if v == x {
            return i
        }
    }
    return -1
}
```

### Instantiation

You can explicitly instantiate a generic function, or let the compiler infer types.

```go
// Explicit instantiation
f := Min[int] 
fmt.Println(f(2, 3))

// Type inference (common usage)
fmt.Println(Min(2, 3)) 
```

### Generic Map/Filter/Reduce

Patterns common in functional programming are now type-safe in Go.

```go
package main

import "fmt"

func Map[T any, R any](s []T, transform func(T) R) []R {
    result := make([]R, len(s))
    for i, v := range s {
        result[i] = transform(v)
    }
    return result
}

func main() {
    nums := []int{1, 2, 3}
    strs := Map(nums, func(n int) string {
        return fmt.Sprintf("Number %d", n)
    })
    fmt.Println(strs)
}
```

### Limitations

*   **No Method Type Parameters**: As noted in "Why Generics", methods cannot have their own type parameters.
*   **Operator Overloading**: Go generics do not support operator overloading. You rely on constraints to define allowable operators.

### Visualizing Constraints

```mermaid
flowchart TD
    A[Function: Min[T]] --> B{Constraint Check}
    B --> C[Constraint: Ordered]
    C --> D{Is int Ordered?}
    C --> E{Is struct Ordered?}
    
    D -->|Yes| F[Compile OK]
    E -->|No| G[Compile Error:<br/>Operator < not defined]
```

## Interview Questions

**Q: What is the `comparable` constraint?**

**A:** `comparable` is a built-in constraint in Go that allows any type that supports the `==` and `!=` operators. It includes booleans, numbers, strings, pointers, channels, interfaces, and arrays/structs of comparable types. It is strictly required for any type parameter used as a map key (e.g., `map[K]V` requires `K comparable`).

**Q: How do you handle operators like `+` or `<` in generic functions?**

**A:** You cannot simply use `+` on `any` type. You must declare a constraint that restricts the type parameter to types that support that operator. For example, to use `<`, the type parameter must be constrained by an interface that includes numbers and strings (often called `Ordered`). To use `+` for arithmetic, you'd constrain to numeric types (`Integer | Float`).

**Q: Can a generic function take different types for the same parameter?**

**A:** No, if you declare `func Add[T any](a, b T)`, both `a` and `b` MUST be the same type. You cannot pass an `int` and a `float64` without explicit conversion. If you need distinct types, you must declare multiple parameters: `func Pair[T1 any, T2 any](a T1, b T2)`.

**Q: What happens if you don't specify constraints?**

**A:** If you use `[T any]`, the compiler assumes nothing about `T`. You can only perform operations valid for *all* types: declaring variables, taking addresses, converting to `interface{}`, assignment, and passing to other functions accepting `any`. You cannot use operators like `+`, `*`, or `<`.
