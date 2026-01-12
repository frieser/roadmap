---
---

## Summary
Passing **Null** leads to the "Billion Dollar Mistake" (NullPointerExceptions), forcing caller code to be littered with defensive checks. Passing **Booleans** as function arguments (Flag Arguments) is a code smell that indicates a function violates the Single Responsibility Principle—it does one thing if `true` and another if `false`.

## Detailed Explanation

### 1. Avoid Passing Null
*   **The Problem**: If a function accepts `null`, every line of that function must be paranoid. If a function returns `null`, the caller must be paranoid.
*   **The Solution**:
    *   **Return Empty Collections**: Return `[]` instead of `null` for lists.
    *   **Null Object Pattern**: Return an object that "does nothing" gracefully (e.g., a `ConsoleLogger` instead of `null` logger).
    *   **Optionals**: Use types like `Option<T>` or `Maybe<T>` (common in Rust/Java/FP) to explicitly state presence/absence.

### 2. Avoid Flag Arguments (Booleans)
*   **The Problem**: A function signature like `render(true)` is confusing. What does `true` mean? Is it `render(isSuite)`? `render(force)`?
*   **The SRP Violation**: `func(bool flag)` implies: `if flag { doA() } else { doB() }`. This function does *two* things.
*   **The Solution**: Split the function. Instead of `render(true)`, create `renderSuite()` and `renderTest()`.

## Go Code Examples

### Handling Null (Nil)
In Go, `nil` is useful/idiomatic for errors and pointers, but dangerous for interfaces.

```go
// BAD: Returning nil slice forces caller to check
func GetUsers() []User {
    if dbFailed {
        return nil 
    }
    return users
}

// GOOD: Return empty slice (Zero Value)
// Callers can safely range over it without crashing or checking for nil
func GetUsers() []User {
    if dbFailed {
        return []User{} // or nil, as range handles nil slices gracefully in Go
    }
    return users
}
```

### The Null Object Pattern in Go
```go
type Logger interface {
    Log(msg string)
}

// Concrete implementation
type FileLogger struct { ... }

// Null Object implementation
type NoOpLogger struct{}
func (n NoOpLogger) Log(msg string) { 
    // Do nothing
}

// Usage: You never have to check 'if logger != nil'
func Process(l Logger) {
    l.Log("Processing...") // Safe even if l is NoOpLogger
}
```

### Flag Arguments vs. Split Functions
```go
// BAD: What does 'true' mean?
func CreateUser(name string, isAdmin bool) {
    u := User{Name: name}
    if isAdmin {
        u.Permissions = All
    } else {
        u.Permissions = Basic
    }
    save(u)
}

// Usage: CreateUser("Bob", true) // Confusing

// GOOD: Explicit Intent
func CreateAdmin(name string) {
    u := User{Name: name, Permissions: All}
    save(u)
}

func CreateGuest(name string) {
    u := User{Name: name, Permissions: Basic}
    save(u)
}

// Usage: CreateAdmin("Bob") // Clear
```

## Interview Questions

**Q: Why is passing `false` to a function considered bad practice?**
**A:** It forces the reader to look up the function definition to understand what the boolean controls. It also suggests the function contains hidden control flow (if/else) that mixes concerns. It is cleaner to have two separate functions with descriptive names.

**Q: How does the Null Object Pattern reduce Cyclomatic Complexity?**
**A:** By removing the need for `if object != nil` checks throughout the codebase. The decision of "what to do when nothing is there" is encapsulated once in the Null Object class, rather than repeated in every client usage.

**Q: In Go, is `nil` always bad?**
**A:** No. Go uses `nil` idiomatically. For example, a `nil` error means "success". A `nil` slice behaves like an empty slice. However, a `nil` interface (where the type is concrete but value is nil) can cause panics if methods are called on it. The key is to handle `nil` gracefully at the boundaries rather than passing it deep into business logic.
