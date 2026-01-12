#Golang
---
---

## Summary

`panic` and `recover` provide a mechanism for handling exceptional situations, similar to exceptions in other languages, but intended for vastly different use cases. `panic` stops ordinary flow, unwinding the stack and running deferred functions. `recover` regains control of a panicking goroutine. The Go idiom is: **use errors for normal failures, use panic only for unrecoverable logic errors or programmer bugs.**

## Detailed Explanation

### Panic

Calling `panic(v)` stops the current function execution. It begins executing `defer` statements in the current stack frame, then moves up the stack doing the same, until the program crashes or `recover` is called.

```go
func check(valid bool) {
    if !valid {
        panic("invalid state: this should never happen")
    }
}
```

### Recover

`recover` is a built-in function that regains control of a panicking goroutine. **It is only useful inside a deferred function.**

*   If called normally (not in defer), it returns `nil` and does nothing.
*   If called during a panic (in defer), it stops the panic sequence, returns the value passed to `panic`, and execution continues normally *after the function that panicked*.

```go
package main

import "fmt"

func safeCall() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()

    fmt.Println("About to panic")
    panic("oops")
    fmt.Println("This line never runs")
}

func main() {
    safeCall()
    fmt.Println("Program continues naturally")
}
```

### Usage Pattern: Convert Panic to Error

A common library pattern is to catch internal panics and convert them to returned errors, preventing the whole app from crashing.

```go
func SafeExecute() (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("internal panic: %v", r)
        }
    }()
    
    // logic that might panic...
    return nil
}
```

### When to Panic?

1.  **Initialization**: `regexp.MustCompile` panics if the regex is invalid. This is acceptable during `init()` or global var initialization because if config is wrong, the app shouldn't start.
2.  **Unreachable Code**: If logic dictates a branch is impossible.
3.  **Programmer Logic Error**: Index out of bounds, nil pointer dereference (runtime does this automatically).

### When NOT to Panic?

*   File not found.
*   Network timeout.
*   Bad user input.
*   **Basically anything that is a runtime condition rather than a code bug.** Use `error` return values for these.

### Scope of Recover

`recover` only works in the **same goroutine** as the panic. A panic in a spawned goroutine cannot be recovered by the parent goroutine; it will crash the entire program.

```go
func main() {
    defer func() { recover() }() // Will NOT catch the panic below
    
    go func() {
        panic("I crash the whole app")
    }()
    
    select{}
}
```

## Interview Questions

**Q: When should you use panic instead of returning an error?**

**A:** Panic should be reserved for truly exceptional conditions where the program cannot continue running safely, or for programmer errors (bugs) like index out of bounds or nil pointer dereferences. It is also acceptable during program initialization (e.g., `init()` functions) if essential configuration is invalid. For all expected failure modes (I/O, validation, network), return an `error`.

**Q: Can you recover from a panic in a different goroutine?**

**A:** No. `recover` only works if called from a deferred function within the *same* goroutine that panicked. If a child goroutine panics and does not recover it internally, the entire Go runtime crashes, taking down all other goroutines with it. This is why top-level functions in goroutines often need their own `defer recover()` block in robust applications.

**Q: What happens if `recover()` is called when there is no panic?**

**A:** It simply returns `nil` and has no effect. This allows you to write defer blocks that conditionally handle panics without needing to know if one occurred beforehand (`if r := recover(); r != nil { ... }`).

**Q: Why is `regexp.MustCompile` allowed to panic?**

**A:** The `Must` prefix is a standard library convention indicating that the function will panic on error instead of returning it. This is useful for initializing global variables where handling an error is impossible (you can't return an error from a global var assignment) and where a failure implies a static bug in the code (a bad regex string literal) that should prevent startup.
