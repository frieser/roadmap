#Golang
---
---

## Summary

`fmt.Errorf` allows you to create formatted error messages, injecting dynamic data (like IDs, filenames, or values) into the error string. Since Go 1.13, it also supports the `%w` verb, which "wraps" an underlying error. This creates a chain of errors, preserving the original cause while adding context, which enables `errors.Is` and `errors.As` to check the underlying chain.

## Detailed Explanation

### Basic Formatting

Just like `fmt.Printf`, but returns an `error`.

```go
func getUser(id int) error {
    if id < 0 {
        return fmt.Errorf("invalid user id: %d", id)
    }
    return nil
}
```

### Error Wrapping with `%w`

The `%w` verb (Wrap) tells Go to wrap the original error.

```go
package main

import (
    "errors"
    "fmt"
    "os"
)

func readFile(filename string) error {
    _, err := os.Open(filename)
    if err != nil {
        // Adds context "open config:" AND keeps the original os.PathError available
        return fmt.Errorf("open config: %w", err)
    }
    return nil
}

func main() {
    err := readFile("missing.txt")
    if err != nil {
        fmt.Println(err) // open config: open missing.txt: no such file or directory
        
        // We can check the underlying cause
        if errors.Is(err, os.ErrNotExist) {
            fmt.Println(">> The file is definitely missing.")
        }
    }
}
```

### `%v` vs `%w`

*   **`%v`**: Formats the error as a string. The original error object is lost (converted to text). The returned error does **not** have an `Unwrap` method.
*   **`%w`**: Wraps the error. The returned error has an `Unwrap()` method returning the original error.

Use `%w` when you want callers to be able to inspect the original cause programmatically. Use `%v` when you want to obscure the implementation details (e.g., returning an error to an external API client).

### Wrapping Only One Error

`fmt.Errorf` only supports one `%w` per call. Using multiple `%w` verbs is a compile-time vetting error or runtime formatting error depending on the toolchain version (Go 1.20 introduced `errors.Join`, but `fmt.Errorf` is generally for single chains).

### Structure of Wrapped Error

`fmt.Errorf` returns a struct that looks roughly like this:

```go
type wrapError struct {
    msg string
    err error // The error passed to %w
}

func (e *wrapError) Error() string { return e.msg }
func (e *wrapError) Unwrap() error { return e.err }
```

## Interview Questions

**Q: What is the purpose of the `%w` verb in `fmt.Errorf`?**

**A:** The `%w` verb wraps an error. It inserts the error's string representation into the message (like `%v`) but also stores the original error inside the returned value. The returned error implements the `Unwrap()` method, allowing functions like `errors.Is` and `errors.As` to traverse the error chain and inspect the underlying cause.

**Q: When should you use `%v` instead of `%w` when formatting an error?**

**A:** Use `%v` when you want to add context to an error string but do **not** want to expose the underlying error to the caller. This effectively "seals" the error, preventing callers from relying on implementation details (like `sql.ErrNoRows` or specific `os.PathError` types). It allows you to change the underlying implementation later without breaking the API contract.

**Q: Can you wrap multiple errors with `fmt.Errorf`?**

**A:** No, `fmt.Errorf` supports only one `%w` verb. If you need to combine multiple errors into a single error (e.g., running multiple validation checks and returning all failures), you should use `errors.Join` (Go 1.20+) or a custom multi-error struct, not `fmt.Errorf`.

**Q: How does the printed output differ between `%w` and `%v`?**

**A:** The string output (the result of `.Error()`) is identical for both `%w` and `%v`. Both insert the string representation of the error. The difference is purely internal: `%w` retains the error object for unwrapping, while `%v` discards it.
