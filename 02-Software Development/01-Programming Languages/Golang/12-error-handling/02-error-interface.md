#Golang
---
---

## Summary

In Go, `error` is a built-in interface type, not a concrete struct. It is defined simply as an interface with a single method: `Error() string`. Any type that implements this method satisfies the `error` interface and can be treated as an error. This flexibility allows developers to create custom error types with rich context (like error codes, timestamps, or field names) while still being usable wherever an `error` is expected.

## Detailed Explanation

### The Definition

The entire definition of `error` in the builtin package is:

```go
type error interface {
    Error() string
}
```

### Implementing the Interface

You can make any struct an error by adding an `Error()` method.

```go
package main

import "fmt"

// Custom error struct
type PathError struct {
    Op   string
    Path string
    Err  string
}

// Implementing the error interface
func (e *PathError) Error() string {
    return fmt.Sprintf("%s %s: %s", e.Op, e.Path, e.Err)
}

// Function returning the custom error as the 'error' interface
func deleteFile(path string) error {
    return &PathError{
        Op:   "delete",
        Path: path,
        Err:  "permission denied",
    }
}

func main() {
    err := deleteFile("/root/secret.txt")
    if err != nil {
        // err is of type 'error', but holds *PathError
        fmt.Println(err) // Output: delete /root/secret.txt: permission denied
    }
}
```

### The "Nil Error" Gotcha

Because `error` is an interface, a nil pointer to a custom error type is **not** the same as a nil interface.

```go
package main

import "fmt"

type MyError struct {
    Msg string
}
func (e *MyError) Error() string { return e.Msg }

func badCheck() error {
    var err *MyError = nil
    // Returning a typed nil pointer inside an interface
    return err 
}

func main() {
    err := badCheck()
    // This is TRUE because the interface has a type (*MyError) even though value is nil
    if err != nil {
        fmt.Println("We have an error... or do we?")
        fmt.Printf("Type: %T, Value: %v\n", err, err)
    }
}
```

**Correct Approach:** Always return explicit `nil` for success.

```go
func goodCheck() error {
    var err *MyError = nil
    if err == nil {
        return nil // Explicitly return nil interface
    }
    return err
}
```

### Rich Custom Errors

Since you control the struct, you can add any data you need.

```go
type ValidationError struct {
    Field string
    Reason string
}
func (e *ValidationError) Error() string {
    return fmt.Sprintf("invalid %s: %s", e.Field, e.Reason)
}

func validate(email string) error {
    if email == "" {
        return &ValidationError{Field: "email", Reason: "cannot be empty"}
    }
    return nil
}
```

### Checking Concrete Types

You can use type assertions to get the data back out.

```go
err := validate("")
if valErr, ok := err.(*ValidationError); ok {
    fmt.Println("Field failed:", valErr.Field)
}
```

## Interview Questions

**Q: What is the `error` type in Go?**

**A:** The `error` type is a built-in interface type. It defines a single method signature: `Error() string`. Any concrete type that implements this method can be used as an `error`. This means errors are just values that satisfy this interface, allowing for polymorphic error handling.

**Q: Why is `var err error = (*MyType)(nil)` not equal to `nil`?**

**A:** An interface value in Go consists of a tuple: `(Type, Value)`. An interface is `nil` only if *both* Type and Value are nil. When you assign a typed nil pointer to an interface variable, the interface becomes `(*MyType, nil)`. Since the Type part is present, the interface value itself is not nil. This is a common bug source; always return explicit `nil` to signal success.

**Q: When should you create a custom error struct instead of using `errors.New`?**

**A:** Use a custom struct when calling code needs to programmatically react to *specific attributes* of the error, not just the error message. For example, if you need to return an HTTP status code, a failing SQL query string, or specific validation fields alongside the error message, a custom struct is appropriate. If you only need a static text message, `errors.New` is sufficient.

**Q: How does `fmt.Println(err)` know how to print the error message?**

**A:** `fmt.Println` (and similar functions) uses reflection to check if the value passed to it satisfies the `error` interface. If it does, it calls the `Error()` method to retrieve the string representation. It also checks for `fmt.Stringer`.
