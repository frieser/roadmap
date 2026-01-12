#Golang
---
---

## Summary

The **Pipeline** pattern connects goroutines so that the output of one is the input of the next. It mimics Unix pipes (`cat | grep | sort`). Each stage of the pipeline is a function that takes an input channel, performs a transformation, and returns an output channel. This allows for modular, streaming data processing where stages run concurrently.

## Detailed Explanation

### Structure

Stage 1 (Generator) -> Channel A -> Stage 2 (Square) -> Channel B -> Stage 3 (Print)

### Implementation

```go
package main

import "fmt"

// Stage 1: Generator
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}

// Stage 2: Transformer (Square)
func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}

func main() {
    // Set up the pipeline
    // 2, 3 -> gen -> sq -> print
    c := gen(2, 3)
    out := sq(c)

    // Stage 3: Consumer
    for n := range out {
        fmt.Println(n) // 4, 9
    }
}
```

### Benefits

1.  **Streaming**: Processing starts immediately. You don't need to load all data into memory first. Stage 2 processes item 1 while Stage 1 is generating item 2.
2.  **Concurrency**: Each stage runs in its own goroutine, utilizing multiple cores automatically.
3.  **Modularity**: Stages are decoupled. You can easily insert a `filter` stage between `gen` and `sq` without changing them.

### Cancellation

Real pipelines need a way to stop early (e.g., error occurring in stage 2). Use a `done` channel or `context`.

```go
func sq(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done: // Upstream signal to stop
                return
            }
        }
    }()
    return out
}
```

## Interview Questions

**Q: Why is closing channels important in a pipeline?**

**A:** Closing the output channel signals to the downstream stage that "no more data is coming." This allows the downstream `range` loop to terminate. If you forget to close a channel, the downstream stage will block forever waiting for input, causing a **goroutine leak** and potentially a deadlock.

**Q: How does a pipeline handle backpressure?**

**A:** Pipelines handle backpressure naturally through **unbuffered channels** (or full buffered channels). If Stage 2 is slow, it won't read from the input channel. This causes Stage 1 to block when trying to send. The slowness propagates upstream, automatically throttling the generator to match the speed of the slowest stage.

**Q: How do you implement a pipeline that processes errors?**

**A:** Standard pipelines usually handle data. To handle errors, the channel type can be a struct `Result { Value T; Err error }`. Each stage checks `in.Err`. If present, it passes it along immediately. If not, it does work. Alternatively, pass a separate error channel alongside, but synchronization becomes complex. The result-object approach is usually cleaner.
