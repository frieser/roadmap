#Golang
---
---

## Summary

Since Go 1.13, standard error handling includes "wrapping" errors to add context (like "db query failed: connection refused"). The `errors` package provides three key functions to work with wrapped errors: `errors.Unwrap` (gets the next error in chain), `errors.Is` (matches a specific error value in the chain), and `errors.As` (matches a specific error *type* in the chain). This replaces older patterns of direct equality checks or type assertions which fail on wrapped errors.

## Detailed Explanation

### The Chain of Errors

When you wrap an error (e.g., `fmt.Errorf("... %w", err)`), you create a linked list.

```mermaid
flowchart LR
    E1[Error: "failed to load config"] -->|Unwrap()| E2[Error: "open file"]
    E2 -->|Unwrap()| E3[Error: "permission denied"]
    E3 -->|Unwrap()| Nil[nil]
```

### errors.Unwrap

Returns the result of calling the `Unwrap()` method on an error, or `nil` if there is no underlying error.

```go
func Unwrap(err error) error
```

*Note: You rarely call this directly. You usually use `Is` or `As`.*

### errors.Is (Value Check)

Checks if *any* error in the chain matches a specific target value (like a Sentinel Error).

**Before (Go < 1.13):** `if err == io.EOF` (Fails if wrapped)
**After (Go 1.13+):** `if errors.Is(err, io.EOF)` (Works if wrapped)

```go
package main

import (
    "errors"
    "fmt"
    "io/fs"
    "os"
)

func main() {
    // Wrapping an error
    err := fmt.Errorf("context: %w", fs.ErrNotExist)

    // Check with Is
    if errors.Is(err, fs.ErrNotExist) {
        fmt.Println("The file does not exist (detected inside wrapper)")
    }
}
```

### errors.As (Type Check)

Checks if *any* error in the chain matches a specific **type**, and if so, assigns it to the target. This replaces type assertions.

**Before:** `if e, ok := err.(*os.PathError); ok` (Fails if wrapped)
**After:** `var e *os.PathError; if errors.As(err, &e)` (Works if wrapped)

```go
package main

import (
    "errors"
    "fmt"
    "io/fs"
    "os"
)

func main() {
    _, err := os.Open("missing-file.txt")
    wrappedErr := fmt.Errorf("app failed: %w", err)

    var pathErr *fs.PathError
    // Check if any error in the chain is of type *fs.PathError
    if errors.As(wrappedErr, &pathErr) {
        fmt.Println("It is a PathError!")
        fmt.Println("Path:", pathErr.Path) // We can access fields now
    }
}
```

### Unwrap Interface

To support unwrapping, a custom error type simply needs to implement:

```go
type Wrapper interface {
    Unwrap() error
}
```

## Interview Questions

**Q: What is the difference between `errors.Is` and `errors.As`?**

**A:** `errors.Is` checks for **value equality**. It is used to check against sentinel errors (specific instances like `io.EOF` or `sql.ErrNoRows`). `errors.As` checks for **type compatibility**. It is used when you want to check if the error is of a specific struct type (like `*os.PathError`) and you want to extract that struct to access its fields. `As` is essentially a "deep type assertion" that searches the entire chain.

**Q: Why should you use `errors.Is(err, io.EOF)` instead of `err == io.EOF`?**

**A:** `err == io.EOF` only checks the top-level error. If `io.EOF` was wrapped (e.g., `fmt.Errorf("read failed: %w", io.EOF)`), the equality check will return `false`. `errors.Is` traverses the chain of wrapped errors and returns `true` if `io.EOF` exists anywhere in that chain. Using `Is` makes your code robust against adding context to errors.

**Q: How do you make a custom error type compatible with `errors.Is`?**

**A:** By default, `errors.Is` falls back to `==`. However, you can customize this by implementing the `Is(target error) bool` method on your custom error type. This allows you to define custom logic for when your error should match another error (e.g., matching a category of errors regardless of specific fields).

**Q: What argument must be passed to `errors.As`?**

**A:** You must pass a **pointer to a pointer** (or a pointer to an interface). For example, if you want to find a `*MyError`, you declare `var myErr *MyError` and pass `&myErr` to `errors.As`. If `errors.As` finds a match, it populates `myErr` with the found error. If you pass the wrong type (e.g., just `myErr` or a non-pointer), `errors.As` will panic.
