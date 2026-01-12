---
---

## Summary
**Cyclomatic Complexity** is a quantitative metric used to measure the complexity of a program. It roughly corresponds to the number of linearly independent paths through a source code. High complexity implies code that is hard to understand, hard to test, and prone to bugs. The goal is to keep this number low (typically < 10 per function).

## Detailed Explanation

### 1. Calculation
Start with 1. Add 1 for every:
*   `if`, `else if`
*   `for`, `foreach`, `while`
*   `case` (in a switch)
*   `&&`, `||` (logical operators)

### 2. The Impact of Complexity
*   **Cognitive Load**: Humans can only hold about 7 items in working memory. A function with complexity 20 requires tracking too many states.
*   **Testing**: Complexity 10 means you need at least 10 unit tests to achieve 100% branch coverage. High complexity makes comprehensive testing impossible.

### 3. Strategies to Reduce Complexity
*   **Guard Clauses**: Return early to avoid nesting.
*   **Polymorphism**: Replace `switch` statements with interfaces/strategies.
*   **Extraction**: Move complex logic blocks into separate helper functions.
*   **Table-Driven Methods**: Use maps or arrays to lookup values instead of long `if-else` chains.

## Go Code Examples

### Guard Clauses (Flattening)
```go
// BAD: Nested "Arrow Code" (Complexity High)
func Process(u *User) error {
    if u != nil {
        if u.IsActive {
            if u.HasPermission {
                // Do work...
                return nil
            } else {
                return errors.New("no permission")
            }
        } else {
            return errors.New("inactive")
        }
    } else {
        return errors.New("nil user")
    }
}

// GOOD: Guard Clauses (Complexity Low)
func Process(u *User) error {
    if u == nil {
        return errors.New("nil user")
    }
    if !u.IsActive {
        return errors.New("inactive")
    }
    if !u.HasPermission {
        return errors.New("no permission")
    }
    
    // Do work...
    return nil
}
```

### Table-Driven Tests (Go Idiom)
Instead of writing complex `if` logic in tests, Go prefers tables. This reduces the complexity of the test code itself.

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        a, b, want int
    }{
        {1, 2, 3},
        {0, 0, 0},
        {-1, 1, 0},
    }
    
    for _, tt := range tests {
        if got := Add(tt.a, tt.b); got != tt.want {
            t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.want)
        }
    }
}
```

## Interview Questions

**Q: What is a "good" Cyclomatic Complexity score?**
**A:** 
*   **1-10**: Simple, low risk, good.
*   **11-20**: Moderate risk, complex. Consider refactoring.
*   **21-50**: High risk, untestable.
*   **50+**: Unmaintainable "Spaghetti code".

**Q: How does Polymorphism reduce Cyclomatic Complexity?**
**A:** A large `switch` statement that checks a type and performs an action is a single function with high complexity. By defining an interface and creating separate classes/structs for each type, the logic is distributed. The complexity of the dispatch is handled by the language's runtime (vtable), reducing the complexity of the user code to 1.

**Q: What tools can I use to measure this?**
**A:** 
*   **Go**: `gocyclo`
*   **JS/TS**: `ESLint` (complexity rule)
*   **Java**: `SonarQube`, `JaCoCo`
