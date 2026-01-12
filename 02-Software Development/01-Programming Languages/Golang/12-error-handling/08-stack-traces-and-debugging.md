#Golang
---
---

## Summary

Debugging Go errors often requires more than just the error message. Stack traces allow you to see the execution path that led to an error or panic. While Go's standard `error` doesn't include a stack trace, panics print one automatically. For normal errors, tools like `runtime/debug` or third-party packages (like `pkg/errors` or modern replacements) allow capturing stack traces to aid in debugging complex production issues.

## Detailed Explanation

### Panic Stack Traces

When a Go program panics, the runtime automatically prints:
1.  The panic message.
2.  The goroutine state (running).
3.  The full stack trace (function calls, filenames, line numbers).

```text
panic: something went wrong

goroutine 1 [running]:
main.doWork(...)
    /path/to/main.go:23
main.main()
    /path/to/main.go:15 +0x40
```

### Printing Stack Trace Manually

You can print the current stack trace at any point using `runtime/debug`.

```go
package main

import (
    "fmt"
    "runtime/debug"
)

func main() {
    fmt.Println(string(debug.Stack()))
}
```

### Adding Stack Traces to Errors

Standard `errors.New` and `fmt.Errorf` do **not** capture stack traces. In production, knowing *where* an error originated is crucial.

#### Approach 1: Third-party Packages (Historical standard)

For years, `github.com/pkg/errors` was the standard.

```go
import "github.com/pkg/errors"

func Do() error {
    // New error with stack trace
    return errors.New("failed") 
    // Or wrapping with stack trace
    return errors.Wrap(err, "context") 
}

func main() {
    err := Do()
    fmt.Printf("%+v", err) // %+v prints the stack trace!
}
```

#### Approach 2: Custom Error with Stack (Modern / Stdlib only)

If you don't want dependencies, you can capture the stack manually, though it's tedious. Most teams prefer using a library or simply relying on logs for context + `fmt.Errorf` wrapping chains to approximate the path.

### Debugging Tools (Delve)

`dlv` (Delve) is the standard debugger for Go.

*   `dlv debug`: Compile and start debugging current package.
*   `break main.main`: Set breakpoint.
*   `continue`, `next`, `step`: execution control.
*   `print var`: Inspect variables.
*   `goroutines`: List all goroutines.

### Inspecting Goroutine Dumps

If a process hangs, you can send `SIGQUIT` (Ctrl+\) to a Go program to force it to dump stack traces of **all** goroutines to stderr and exit. This is invaluable for debugging deadlocks.

```bash
$ ./my-app
# (Press Ctrl+\)
SIGQUIT: quit
PC=0x45d...
goroutine 1 [running]:
...
goroutine 17 [select]:
...
```

## Interview Questions

**Q: Do Go errors automatically contain stack traces?**

**A:** No. Standard Go errors (`errors.New`, `fmt.Errorf`) are simple values containing only a string (and potentially a wrapped error). They do not capture the call stack. This keeps errors lightweight. If you need stack traces for debugging, you must use a library (like `pkg/errors`) or explicitly capture the stack using `runtime/debug.Stack()` when the error is created.

**Q: How can you obtain a stack trace of all running goroutines without killing the program?**

**A:** You can use `pprof`. Specifically, the `/debug/pprof/goroutine?debug=1` (or `debug=2` for full stacks) endpoint provided by `net/http/pprof` allows you to view the stack traces of all goroutines in a running application via a browser or curl, without stopping the application.

**Q: What is `SIGQUIT` used for in Go applications?**

**A:** Sending the `SIGQUIT` signal (often via Ctrl+\ in a terminal) causes the Go runtime to print a stack trace of every currently running goroutine to stderr and then exit. This is a built-in feature of the Go runtime, distinct from a panic, and is extremely useful for diagnosing deadlocks or "stuck" processes in development.

**Q: What is Delve?**

**A:** Delve (`dlv`) is the official debugger for the Go programming language. Unlike GDB, Delve understands Go's runtime model (goroutines, stacks, interfaces) and provides a much better debugging experience. It allows setting breakpoints, stepping through code, inspecting variables, and attaching to running processes.
