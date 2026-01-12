#Golang
---
---

## Summary

`sync.Mutex` provides mutual exclusion locking to protect shared memory from concurrent access (Data Races). It is the fundamental primitive for "shared memory concurrency" in Go. `sync.RWMutex` is a reader/writer lock that allows multiple concurrent readers but only one writer, improving performance for read-heavy workloads.

## Detailed Explanation

### sync.Mutex

Ensures only one goroutine executes a critical section at a time.

```go
package main

import (
    "fmt"
    "sync"
)

type SafeCounter struct {
    mu    sync.Mutex
    value int
}

func (c *SafeCounter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock() // Ensure unlock happens even if panic
    c.value++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}
```

### sync.RWMutex

Optimized for scenarios where reads happen much more often than writes.

*   `RLock()`: Read lock. Blocks if a Write lock is held. Allows other Read locks.
*   `Lock()`: Write lock. Blocks until **all** readers and writers release locks.

```go
type SafeMap struct {
    mu   sync.RWMutex
    data map[string]string
}

func (m *SafeMap) Get(key string) string {
    m.mu.RLock() // Multiple readers allowed
    defer m.mu.RUnlock()
    return m.data[key]
}

func (m *SafeMap) Set(key, value string) {
    m.mu.Lock() // Exclusive access
    defer m.mu.Unlock()
    m.data[key] = value
}
```

### Channels vs Mutexes

> "Share memory by communicating, don't communicate by sharing memory." - Rob Pike

*   **Use Channels**: Passing ownership of data, distributing units of work, communicating async results.
*   **Use Mutexes**: Caches, state (like counters), struct fields that need atomic updates, performance-critical sections (mutex is faster than channel).

### Common Pitfalls

1.  **Copying Mutexes**: A `sync.Mutex` must **not** be copied after first use. Always pass structs containing mutexes by **pointer**.
2.  **Deadlocks**: Locking `mu` twice in the same goroutine without unlocking.
3.  **Unlock placement**: Always use `defer Unlock()` immediately after `Lock()` to prevent bugs where early returns leave the mutex locked.

## Interview Questions

**Q: What is the difference between `Lock()` and `RLock()` in `sync.RWMutex`?**

**A:** `Lock()` is an exclusive write lock: no other goroutine can read (`RLock`) or write (`Lock`) until it is released. `RLock()` is a shared read lock: it allows other goroutines to also acquire `RLock` simultaneously, but blocks any goroutine trying to acquire `Lock`. Use `RWMutex` when you have many readers and few writers to increase concurrency.

**Q: Why should you never copy a `sync.Mutex`?**

**A:** A `sync.Mutex` contains internal state (flags, semaphore pointers) that tracks the lock status. If you copy a struct containing a mutex (e.g., passing by value), you copy that internal state. The copy is a *new, separate* mutex with the old state, detached from the original. Locking the copy won't protect the original data, rendering the lock useless and potentially causing deadlocks or data races. Always use pointers.

**Q: When would you choose a Mutex over a Channel?**

**A:** Use a Mutex when guarding internal state of a struct (like a map or counter) where the critical section is very short and simple. Mutexes are generally lighter weight and faster for fine-grained locking. Use Channels when you need to coordinate execution flow, pass data ownership between goroutines, or implement worker pools.

**Q: Does `RWMutex` prevent writer starvation?**

**A:** Yes, the Go `sync.RWMutex` implementation prioritizes writers. If a writer calls `Lock()`, generally no new readers are granted `RLock()` until the writer has finished, preventing a stream of constant readers from blocking the writer indefinitely.
