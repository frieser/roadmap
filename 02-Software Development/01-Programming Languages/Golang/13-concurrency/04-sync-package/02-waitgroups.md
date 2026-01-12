#Golang
---
---

## Summary

`sync.WaitGroup` is a synchronization primitive used to wait for a collection of goroutines to finish. It works like a countdown latch. The main goroutine sets the number of tasks, starts the workers, and then blocks until the counter reaches zero. It is the standard way to synchronize the completion of parallel tasks in Go.

## Detailed Explanation

### Core Methods

1.  `Add(delta int)`: Increments the counter by `delta`. Usually `Add(1)` before starting a goroutine.
2.  `Done()`: Decrements the counter by 1. Usually called via `defer` inside the goroutine.
3.  `Wait()`: Blocks until the counter becomes 0.

### Basic Pattern

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done() // 3. Signal completion
    
    fmt.Printf("Worker %d starting\n", id)
    time.Sleep(time.Second)
    fmt.Printf("Worker %d done\n", id)
}

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 3; i++ {
        wg.Add(1) // 1. Add before starting
        go worker(i, &wg) // Pass by pointer!
    }

    wg.Wait() // 2. Block until all done
    fmt.Println("All workers finished")
}
```

### Important Rules

1.  **Add before Go**: Always call `wg.Add(1)` *before* the `go` statement. If you put it inside the goroutine, `wg.Wait()` might run (and return) before the goroutine has a chance to start and increment the counter.
2.  **Pass by Pointer**: Like Mutexes, WaitGroups must **not be copied**. Always pass `*sync.WaitGroup` to functions.
3.  **Panic Safety**: Use `defer wg.Done()` to ensure the counter is decremented even if the worker panics.

### WaitGroup vs Channels

*   **WaitGroup**: Use when you just need to wait for completion (signal "all done").
*   **Channels**: Use when you need to collect results/data from the workers while waiting.

### Visual Flow

```mermaid
flowchart TD
    Main[Main Goroutine] -->|Add(3)| WG[WaitGroup Counter: 3]
    Main --> Start1[Go Worker 1]
    Main --> Start2[Go Worker 2]
    Main --> Start3[Go Worker 3]
    Main -->|Wait()| Block[Blocked...]
    
    Start1 -->|Done()| WG
    Start2 -->|Done()| WG
    Start3 -->|Done()| WG
    
    WG -->|Counter == 0| Block
    Block --> Continue[Main Continues]
```

## Interview Questions

**Q: Why must you pass `sync.WaitGroup` by pointer?**

**A:** `sync.WaitGroup` maintains internal state (the counter). If passed by value, a copy is created with its own independent counter. The worker would decrement the copy, while the main goroutine waits on the original, which never reaches zero. This leads to a **deadlock**.

**Q: Where should `wg.Add()` be called? Inside or outside the goroutine?**

**A:** Outside. Specifically, immediately before the `go` statement. If called inside, there is a race condition: the scheduler might execute `wg.Wait()` in the main thread before the new goroutine gets CPU time to call `wg.Add()`. If the counter is 0, `Wait()` returns immediately, and the program exits prematurely.

**Q: How can `WaitGroup` cause a panic?**

**A:** If the counter goes negative (i.e., you call `Done()` more times than `Add()`), `sync.WaitGroup` will panic. This usually happens if you have logic errors in your workers or if you reuse a WaitGroup incorrectly without waiting for it to finish first.

**Q: Can you reuse a `sync.WaitGroup`?**

**A:** Yes, but only after `Wait()` has returned. Once the counter hits zero and releases all waiters, it is ready to be used again. However, concurrent calls to `Add()` while `Wait()` is active trigger a race condition panic.
