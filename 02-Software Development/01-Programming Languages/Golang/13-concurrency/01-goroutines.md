#Golang
---
---

## Summary

Goroutines are lightweight threads managed by the Go runtime. They allow functions to run concurrently with minimal overhead. Created using the `go` keyword, goroutines share the same address space but have their own growing stacks. Thousands of goroutines can run on a single OS thread, multiplexed by the Go scheduler (M:N scheduling), making Go ideal for high-concurrency applications.

## Detailed Explanation

### Basic Syntax

```go
go functionName(arguments)

go func() {
    // anonymous function body
}()
```

### Simple Example

```go
package main

import (
    "fmt"
    "time"
)

func say(s string) {
    for i := 0; i < 3; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}

func main() {
    // Start goroutine
    go say("world")
    
    // Run function in main goroutine
    say("hello")
    
    // Output will be interleaved (non-deterministic)
}
```

### Lightweight Nature

Goroutines start with a tiny stack (~2KB), which grows and shrinks dynamically. OS threads typically start with large fixed stacks (e.g., 1MB). This allows Go programs to spawn hundreds of thousands of goroutines without exhausting system memory.

### Go Runtime Scheduler (M:N)

The Go runtime multiplexes **M** goroutines onto **N** OS threads.

*   **G (Goroutine)**: The code to execute.
*   **M (Machine)**: The OS thread.
*   **P (Processor)**: A resource required to execute Go code (default = number of CPU cores).

When a goroutine blocks (e.g., I/O, channel wait), the scheduler parks it and runs another goroutine on the same OS thread, efficiently utilizing CPU time.

### Main Goroutine

The `main` function runs in its own goroutine (the "main goroutine"). When `main` returns, the program exits immediately, terminating all other running goroutines, even if they haven't finished.

```go
package main

import "fmt"

func main() {
    go func() {
        fmt.Println("I might not print")
    }()
    // No sleep or wait - program exits immediately
}
```

To wait for goroutines, use synchronization primitives like `sync.WaitGroup` or channels.

### Goroutines vs Threads

| Feature | Goroutine | OS Thread |
| :--- | :--- | :--- |
| **Startup Size** | ~2KB (growable) | ~1MB (fixed) |
| **Management** | Go Runtime (User space) | OS Kernel |
| **Switch Cost** | Low (few instructions) | High (context switch) |
| **ID** | No public ID | Thread ID |
| **Communication**| Channels (CSP) | Shared Memory |

### Closure Capture Gotcha

A common bug when launching goroutines in a loop:

```go
// BUG: Captures loop variable by reference
for i := 0; i < 5; i++ {
    go func() {
        fmt.Println(i) // Likely prints "5" five times
    }()
}

// FIX: Pass as argument (creates copy)
for i := 0; i < 5; i++ {
    go func(val int) {
        fmt.Println(val)
    }(i)
}
```
*(Note: Go 1.22 fixed this specific loop variable behavior, but understanding capture is still important for older codebases and other contexts.)*

## Interview Questions

**Q: What is the difference between concurrency and parallelism in Go?**

**A:** Concurrency is about **structure**: dealing with many things at once (breaking a program into independently executing pieces). Parallelism is about **execution**: doing many things at once (running on multiple CPU cores simultaneously). Goroutines enable concurrency. If you have multiple cores (`GOMAXPROCS > 1`), the runtime executes them in parallel. "Concurrency is not parallelism" is a famous talk by Rob Pike explaining this distinction.

**Q: How does the Go scheduler work?**

**A:** Go uses an M:N scheduler, multiplexing M goroutines onto N OS threads. It uses a "work-stealing" algorithm. Each P (Processor) has a local run queue of goroutines. If a P runs out of work, it tries to steal half the goroutines from another P's queue. This balances load across cores efficiently without a central lock bottleneck.

**Q: Why doesn't a Go program wait for all goroutines to finish?**

**A:** The language spec dictates that when the `main` function returns, the program exits. It does not wait for other goroutines. This design prevents "zombie" processes kept alive by forgotten background tasks. If you need to wait, you must explicitly synchronize using `sync.WaitGroup` or channels to block `main` until work is complete.

**Q: What is the cost of a goroutine context switch compared to a thread context switch?**

**A:** A goroutine switch is much cheaper (nanoseconds vs microseconds). It only involves swapping 3 registers (PC, SP, DX) and is handled in user space. An OS thread switch requires entering kernel mode, flushing TLB caches, and saving/restoring all CPU registers (AVX/SSE), which is significantly more expensive.
