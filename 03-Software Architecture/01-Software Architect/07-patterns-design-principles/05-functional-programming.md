---
---

## Summary
**Functional Programming (FP)** is a paradigm where programs are constructed by applying and composing functions. It emphasizes **immutability**, **pure functions**, and **declarative code**. For a Software Architect, FP offers a way to build systems that are easier to reason about, test, and parallelize (concurrency-friendly).

## Detailed Explanation

### 1. Core Concepts
*   **Pure Functions**: A function where the return value is determined *only* by its input values, without observable side effects (no global state, no I/O inside).
    *   *Benefit*: Trivial to test and memoize.
*   **Immutability**: Once a variable is created, it cannot be changed. Instead of modifying an object, you create a new one with the updated value.
    *   *Benefit*: Thread-safe by default. No race conditions on read-only data.
*   **Higher-Order Functions**: Functions that take other functions as arguments or return them as results.
    *   *Benefit*: Enables powerful abstractions (Map, Filter, Reduce).
*   **Referential Transparency**: An expression can be replaced by its value without changing the program's behavior.

### 2. FP vs. OOP
*   **OOP**: Encapsulates moving parts (state) and allows code access to it. "Object represents a noun."
*   **FP**: Minimizes moving parts. Separates data from behavior. "Function represents a verb."

### 3. Monads & Functors (Briefly)
While deep theory, practical FP uses:
*   **Functor**: Something that can be mapped over (like a List or Option).
*   **Monad**: A design pattern to handle "side effects" (like `Maybe`, `Promise`, or `IO`) in a pure way, often allowing chaining operations.

---

## Go Application (FP Style)

Go is not a pure functional language (it has pointers and mutation), but supports FP features like First-Class Functions and Closures.

### Pure Functions & Higher-Order Functions

```go
package main

import "fmt"

// Pure Function: Output depends only on Input. No side effects.
func add(a, b int) int {
	return a + b
}

// Higher-Order Function: Takes a function 'op' as an argument
func compute(a, b int, op func(int, int) int) int {
	return op(a, b)
}

// Map: Implementing a common FP pattern
func Map(data []int, f func(int) int) []int {
	result := make([]int, len(data))
	for i, v := range data {
		result[i] = f(v)
	}
	return result
}

func main() {
	nums := []int{1, 2, 3, 4}

	// Using Map with an anonymous function (Closure)
	doubled := Map(nums, func(x int) int {
		return x * 2
	})

	fmt.Println(doubled) // [2 4 6 8]
}
```

### Immutability (Simulated)
In Go, we pass by value to simulate immutability.

```go
type User struct {
    Name string
    Age  int
}

// Returns a NEW User instead of modifying the pointer
func UpdateAge(u User, newAge int) User {
    u.Age = newAge // Modifies the copy
    return u
}
```

---

## Interview Questions

**Q: What is a Pure Function and why is it useful in distributed systems?**
**A:** A pure function has no side effects and always produces the same output for the same input. In distributed systems, this is crucial for **Idempotency** and **Caching**. If a function is pure, its result can be safely cached forever, and retrying the function (on network failure) won't corrupt the system state.

**Q: Explain "Side Effects" in the context of FP.**
**A:** A side effect is anything a function does other than returning a value. Examples: Modifying a global variable, writing to a file, printing to console, or making an API call. FP aims to isolate these side effects to the "edges" of the system (e.g., in the `main` function or specific I/O layers) to keep the core logic pure.

**Q: How does Immutability help with Concurrency?**
**A:** Race conditions happen when two threads try to modify the same memory location simultaneously. If data is immutable, it cannot be modified after creation. Therefore, multiple threads can read the same data without locks or synchronization overhead, significantly simplifying concurrent programming.
