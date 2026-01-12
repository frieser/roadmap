#Golang
---
---

## Summary

The `context` package is standard in Go for carrying deadlines, cancellation signals, and request-scoped values across API boundaries and goroutines. It allows you to propagate a "stop" signal down a call chain. If a user cancels a request or a timeout is reached, all goroutines working on that request should stop immediately to save resources.

## Detailed Explanation

### The Context Interface

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key any) any
}
```

### Creating Contexts

1.  `context.Background()`: Root context, empty. Used in main, init, tests.
2.  `context.TODO()`: Placeholder when you don't know which context to use yet.
3.  `context.WithCancel(parent)`: Returns a copy that closes `Done` channel when `cancel()` is called.
4.  `context.WithTimeout(parent, duration)`: Cancels after duration.
5.  `context.WithDeadline(parent, time)`: Cancels at specific time.

### Cancellation Pattern

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done(): // Check for cancellation
            fmt.Println("Worker stopped:", ctx.Err())
            return
        default:
            fmt.Println("Working...")
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    // Create context that can be cancelled
    ctx, cancel := context.WithCancel(context.Background())
    
    go worker(ctx)
    
    time.Sleep(2 * time.Second)
    fmt.Println("Cancelling context...")
    cancel() // Signal worker to stop
    
    time.Sleep(1 * time.Second)
}
```

### Timeout Pattern

Crucial for network requests to prevent hanging forever.

```go
func longRunningOp(ctx context.Context) error {
    select {
    case <-time.After(5 * time.Second):
        return nil // Completed
    case <-ctx.Done():
        return ctx.Err() // "context deadline exceeded"
    }
}

func main() {
    // Automatically cancels after 2 seconds
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel() // Best practice: always call cancel to release resources

    if err := longRunningOp(ctx); err != nil {
        fmt.Println("Operation failed:", err)
    }
}
```

### Best Practices

1.  **First Argument**: Pass `ctx context.Context` as the first argument to functions.
2.  **Don't store in structs**: Context should flow through function calls, not be stored in objects.
3.  **Cancel functions**: Always call the `cancel` function returned by `WithCancel`/`WithTimeout`, usually via `defer`, to prevent context leaks.
4.  **Immutable**: Contexts are immutable. `WithCancel` returns a *child* context; it doesn't modify the parent.

## Interview Questions

**Q: Why should `context.Context` always be the first argument?**

**A:** It is a strong convention in Go. It makes it immediately obvious which functions are context-aware (cancellable/scoped) and ensures consistency across libraries. It helps in readability and maintainability, ensuring context propagation is explicit.

**Q: What is a "context leak" and how do you prevent it?**

**A:** When you create a context with `WithTimeout` or `WithCancel`, the runtime creates internal timers and goroutines to manage that state. If you don't call the returned `cancel()` function, these resources linger until the *parent* context is cancelled (which might be never for `Background`). Preventing it is simple: `defer cancel()` immediately after creation.

**Q: What happens to child contexts when a parent context is cancelled?**

**A:** Cancellation propagates downwards. When a parent context is cancelled, all of its children (and grandchildren) are automatically cancelled immediately. This is designed for request scoping: if the main HTTP request is cancelled, all database queries and helper goroutines spawned for that request should also stop.

**Q: Should you store Context in a struct?**

**A:** Generally, no. Context is request-scoped, while structs often have a different lifecycle. Storing context in a struct can lead to confusion about *which* context applies to a method call and can make objects hard to reuse. The exception is when the struct *is* the request (like `http.Request`), where the context is intimately tied to that specific object's lifecycle.
