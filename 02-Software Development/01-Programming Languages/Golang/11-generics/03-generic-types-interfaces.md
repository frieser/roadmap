#Golang
---
---

## Summary

Just as functions can be generic, Go allows defining **Generic Types**. These are typically struct or interface definitions that take type parameters. This enables creating reusable data structures like linked lists, stacks, queues, or trees that work with any data type while maintaining type safety. Interfaces in Go 1.18+ also evolved to define "Type Sets", serving as constraints for generics.

## Detailed Explanation

### Generic Structs

Syntax matches generic functions: `type Name[T Constraint] struct { ... }`.

#### Example: Generic Stack

```go
package main

import "fmt"

// Stack is a generic struct
type Stack[T any] struct {
    elements []T
}

// Method on generic type
// Note: Receiver specifies type parameter [T]
func (s *Stack[T]) Push(v T) {
    s.elements = append(s.elements, v)
}

func (s *Stack[T]) Pop() (T, bool) {
    var zero T
    if len(s.elements) == 0 {
        return zero, false
    }
    index := len(s.elements) - 1
    element := s.elements[index]
    s.elements = s.elements[:index]
    return element, true
}

func main() {
    // Instantiate with int
    intStack := Stack[int]{}
    intStack.Push(10)
    
    // Instantiate with string
    strStack := Stack[string]{}
    strStack.Push("Hello")
    
    fmt.Println(intStack)
    fmt.Println(strStack)
}
```

### Generic Linked List

```go
type Node[T any] struct {
    Value T
    Next  *Node[T]
}

type List[T any] struct {
    Head *Node[T]
}

func (l *List[T]) Add(val T) {
    newNode := &Node[T]{Value: val}
    if l.Head == nil {
        l.Head = newNode
        return
    }
    current := l.Head
    for current.Next != nil {
        current = current.Next
    }
    current.Next = newNode
}
```

### Interfaces as Constraints (Type Sets)

In Go 1.18, interfaces became more powerful. They can now define **sets of types**, not just sets of methods.

#### Basic Constraint

```go
// Number interface allows int or float64
type Number interface {
    int | float64
}

// T must satisfy Number (be int or float64)
func Add[T Number](a, b T) T {
    return a + b
}
```

#### Mixed Constraint (Methods + Types)

```go
// Stringer interface (standard)
type Stringer interface {
    String() string
}

// PrintableNum requires being an int AND implementing String()
type PrintableNum interface {
    int
    String() string
}
```

### The `~` Tilde Token

The tilde `~` allows underlying types to match.

```go
type MyInt int

// Without ~, MyInt would NOT satisfy this
type Integer interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64
}

// Now MyInt satisfies Integer because its underlying type is int
func Double[T Integer](val T) T {
    return val * 2
}
```

### Generic Interfaces

Interfaces themselves can have type parameters.

```go
package main

// A generic interface
type Container[T any] interface {
    Add(T)
    Get(int) T
}

// A struct implementing the generic interface
type Box[T any] struct {
    item T
}

func (b *Box[T]) Add(item T) { b.item = item }
func (b *Box[T]) Get(i int) T { return b.item }

func main() {
    // Variable of interface type
    var c Container[int] = &Box[int]{}
    c.Add(5)
}
```

### Composition of Constraints

Constraints can embed other interfaces.

```go
import "golang.org/x/exp/constraints"

type Signed interface {
    constraints.Signed
}

type Unsigned interface {
    constraints.Unsigned
}

// Union of interfaces
type AnyInteger interface {
    Signed | Unsigned
}
```

### Visualization: Type Sets

```mermaid
flowchart TD
    subgraph "Interface: Number"
        A[int]
        B[float64]
        C[MyFloat (type MyFloat float64)]
    end
    
    D[Constraint definition] -->|int | ~float64| E[Type Set]
    
    A -->|Matches int| E
    B -->|Matches ~float64| E
    C -->|Matches ~float64| E
    
    F[string] -->|Does not match| E
```

## Interview Questions

**Q: Can you define a method on a generic type with a DIFFERENT type parameter?**

**A:** No. Methods on a generic type `Stack[T]` automatically use `T`. You cannot define `func (s *Stack[T]) Map[R any](...)`. If you need a different type parameter, you must define a top-level function taking the stack as an argument: `func Map[T, R any](s Stack[T], f func(T) R) Stack[R]`.

**Q: What is the `~` operator in an interface definition?**

**A:** The tilde `~` operator (e.g., `~int`) specifies that the constraint includes any type whose **underlying type** is `int`. Without `~`, `int` only matches the exact type `int`. With `~int`, it matches `int`, `type MyInt int`, `type UserId int`, etc. This is crucial for creating useful libraries that work with custom types.

**Q: How do generic structs affect memory layout?**

**A:** They don't have a layout until instantiated. When you declare `Stack[int]`, the compiler generates a struct layout equivalent to `struct { elements []int }`. `Stack[bool]` generates `struct { elements []bool }`. They are distinct types with distinct memory layouts, optimized for the specific type (e.g., proper alignment/padding).

**Q: Can a generic type be used as a type assertion?**

**A:** No. You cannot perform type assertions or type switches on generic types at runtime because the type parameter information is not fully available in the same way for dynamic dispatch. For example, `x.(T)` where `T` is a type parameter is allowed, but asserting *to* a generic instance `x.(List[int])` requires care and isn't always supported in all contexts depending on the compiler version and exact usage.
