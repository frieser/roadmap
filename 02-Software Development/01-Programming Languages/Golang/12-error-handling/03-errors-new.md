#Golang
---
---

## Summary

`errors.New` is the most basic way to create an error in Go. It creates a simple error with a static string message. It is part of the standard `errors` package. Because Go errors are values, `errors.New` returns a pointer to a struct, ensuring that each call creates a unique error instance, even if the text is identical.

## Detailed Explanation

### Basic Usage

```go
package main

import (
    "errors"
    "fmt"
)

func validateAge(age int) error {
    if age < 0 {
        return errors.New("age cannot be negative")
    }
    return nil
}

func main() {
    err := validateAge(-5)
    if err != nil {
        fmt.Println(err) // age cannot be negative
    }
}
```

### Equality and Identity

Each call to `errors.New` returns a distinct memory address.

```go
package main

import (
    "errors"
    "fmt"
)

func main() {
    err1 := errors.New("something failed")
    err2 := errors.New("something failed")

    // Comparison checks pointers (identity)
    if err1 == err2 {
        fmt.Println("Same error")
    } else {
        fmt.Println("Different errors") // This prints
    }
}
```

This behavior is important for **Sentinel Errors** (predefined global error variables), as it ensures that `io.EOF` is unique and cannot be spoofed by creating a new error with the text "EOF".

### Implementation (Simplification)

Under the hood, `errors.New` looks something like this:

```go
package errors

func New(text string) error {
    return &errorString{text}
}

// Unexported struct prevents type assertions matching it elsewhere
type errorString struct {
    s string
}

func (e *errorString) Error() string {
    return e.s
}
```

### When to Use

Use `errors.New`:
*   When you need a simple, static error message.
*   When defining package-level sentinel errors (e.g., `var ErrNotFound = errors.New("not found")`).
*   When you don't need to format data into the string (use `fmt.Errorf` for that).

## Interview Questions

**Q: What is the difference between `errors.New("fail")` and `fmt.Errorf("fail")`?**

**A:** Functionally, if there are no format verbs, they produce similar results. However, `errors.New` is cheaper as it doesn't need to parse a format string. `fmt.Errorf` allows you to format dynamic data into the error message (e.g., `fmt.Errorf("user %d not found", id)`) and, significantly, allows wrapping errors using the `%w` verb.

**Q: Why does `errors.New` return a pointer?**

**A:** It returns a pointer to ensure uniqueness. If it returned a value (struct), errors with the same message would compare as equal (`struct{s string}` is comparable by value). By returning a pointer, `errors.New("EOF") != errors.New("EOF")`. This allows sentinel errors to work based on identity, not just string content.

**Q: Can I modify the message of an error created by `errors.New`?**

**A:** No. The struct returned by `errors.New` is unexported (`errors.errorString`), and its field is unexported. There is no public way to modify the string after creation. Errors in Go are generally intended to be immutable.

**Q: Is `errors.New` safe for concurrent use?**

**A:** Yes. Since the created error is effectively immutable (you can't change the internal string), sharing the same error instance (like a global `ErrNotFound`) across multiple goroutines is perfectly safe.
