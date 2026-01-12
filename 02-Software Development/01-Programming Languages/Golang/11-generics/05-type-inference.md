#Golang
---
---

## Summary

Type inference allows omitting type arguments when calling generic functions. The Go compiler deduces the type parameters from the arguments passed to the function. This makes generic code feel like regular Go code, reducing verbosity. Inference works for both simple types and complex structural types (e.g., inferring slice element types).

## Detailed Explanation

### Basic Inference

```go
func Min[T constraints.Ordered](a, b T) T { ... }

func main() {
    // Explicit
    x := Min[int](2, 3)

    // Inferred: Compiler sees 2 and 3 are ints -> T is int
    y := Min(2, 3) 
}
```

### Structural Inference

The compiler can look inside composite types (slices, maps, functions) to infer the type parameter.

```go
func Map[T, R any](s []T, f func(T) R) []R { ... }

func main() {
    nums := []int{1, 2, 3}
    
    // T is inferred as int (from nums)
    // R is inferred as string (from the return type of the anonymous function)
    result := Map(nums, func(n int) string {
        return fmt.Sprint(n)
    })
    
    // No need to write: Map[int, string](...)
}
```

### When Inference Fails

Inference fails if the type cannot be determined from arguments alone, or if arguments imply conflicting types.

1.  **Arguments don't match**:
    ```go
    Min(1, 2.5) // Error: T cannot be both int and float64
    // Fix: Min[float64](1, 2.5) // Explicitly set T, 1 converts to 1.0
    ```

2.  **No arguments**:
    ```go
    func Zero[T any]() T { var z T; return z }
    
    // z := Zero() // Error: cannot infer T
    z := Zero[int]() // Must be explicit
    ```

3.  **Return type only**:
    If the type parameter is used *only* for the return type and not in the arguments, inference isn't possible from the call site context (Go doesn't support return-type-based inference like Haskell/Rust in this way).

### Partial Inference

Go does not currently support partial type argument inference (supplying one but inferring another). You must supply all or none.

```go
// func F[A, B any](a A) B
// F[int](5) // Error: got 1 type argument, want 2
```

### Type Unification

The compiler performs **unification** to find a type that satisfies all usages.

```mermaid
flowchart LR
    A[Call: Min(a, b)]
    B[Arg a: int]
    C[Arg b: int]
    D[Parameter: T]
    
    B -->|Unify T = int| D
    C -->|Unify T = int| D
    D -->|Success| E[T is int]
    
    F[Arg a: int]
    G[Arg b: float64]
    F -->|Unify T = int| H[T]
    G -->|Unify T = float64| H
    H -->|Conflict| I[Compile Error]
```

### Inference with Untyped Constants

Untyped constants (literals) provide flexibility.

```go
// Min[T](a, b T)
Min(1, 2.0) // Error: int vs float64

// But if one is a variable and one is a literal:
var f float64 = 2.0
Min(f, 1) // OK! '1' is untyped constant, can be float64. T inferred as float64.
```

## Interview Questions

**Q: Can Go infer type arguments based on the variable I assign the result to?**

**A:** No. Go type inference flows from the *arguments* to the *type parameters*. It does not flow backwards from the expected result type.
Example: `var x int = Zero()` will fail. You must write `var x int = Zero[int]()`. This design choice keeps the type checker simple and predictable (context-independent).

**Q: What happens if I pass an `int` and a `float64` to a generic function `func Add[T Number](a, b T)`?**

**A:** Compilation error. Go inference requires the type parameter `T` to resolve to a single concrete type. `int` and `float64` are distinct. Even though both satisfy `Number`, they don't match *each other*. You must explicitly cast one argument: `Add(float64(myInt), myFloat)` or explicitly instantiate `Add[float64](...)`.

**Q: Does inference work with custom types?**

**A:** Yes. If you have `type MyInt int` and pass it to `Min`, `T` will be inferred as `MyInt`, provided the constraint allows it (usually requires the `~int` approximation).

**Q: Why can't I provide just one type argument if a function has two?**

**A:** Go does not support partial instantiation. If a generic function has multiple type parameters, you must either let the compiler infer *all* of them, or you must explicitly provide *all* of them.
