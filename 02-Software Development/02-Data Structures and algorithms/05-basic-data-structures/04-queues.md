---
---

# Queues

## Summary
A **Queue** is a linear data structure that follows the **FIFO** (First-In, First-Out) principle. Elements are added at the rear (Enqueue) and removed from the front (Dequeue). Go offers multiple ways to implement queues: Slices (simple), `container/list` (stable memory), or Channels (concurrent).

## Detailed Explanation

### 1. Operations
*   **Enqueue**: Add element to the tail.
*   **Dequeue**: Remove element from the head.
*   **Peek**: View the head element.

### 2. Implementation Strategies in Go
*   **Slice-Based**: Simplest, but suffers from "memory drift" (the underlying array grows but the active window moves forward, potentially wasting memory unless re-sliced/copied).
*   **Linked List**: Uses `container/list`. Stable memory usage, no resizing needed, but higher per-node overhead.
*   **Channel**: Built-in thread-safe queue. Ideal for producer-consumer patterns.

## Code Examples (Go)

### 1. Slice Implementation (Basic)
Suitable for short-lived queues or algorithms like BFS.

```go
package main

import "fmt"

func main() {
    queue := []string{}

    // Enqueue
    queue = append(queue, "Job 1")
    queue = append(queue, "Job 2")

    // Dequeue
    if len(queue) > 0 {
        front := queue[0]
        queue = queue[1:] // Shift slice window
        fmt.Println("Processed:", front)
    }
}
```

### 2. Linked List Implementation (Robust)
Better for long-running queues to avoid slice memory issues.

```go
import "container/list"

func main() {
    q := list.New()
    
    // Enqueue
    q.PushBack("Job 1")
    
    // Dequeue
    if front := q.Front(); front != nil {
        val := q.Remove(front)
        fmt.Println(val)
    }
}
```

## Go Application: Priority Queues
For a queue where items are processed based on importance rather than arrival time, Go provides `container/heap`.

```go
// See 'container/heap' documentation for the full Interface implementation
// Core logic:
// Push: Add to slice, then 'up-heap' (bubble up)
// Pop: Swap root with last, remove last, then 'down-heap' (bubble down)
```

## Interview Questions

**Q: What is the downside of using `queue = queue[1:]` to dequeue in Go?**
**A:** The underlying array remains allocated. If you process 1 million items but only keep 10 in the queue, the array might still grow to size 1,000,000 and never shrink, leading to a memory leak. You occasionally need to copy the active elements to a new, smaller slice.

**Q: How do you implement a thread-safe Queue in Go?**
**A:** The most idiomatic way is to use a **Buffered Channel** (`make(chan T, size)`). Alternatively, protect a slice with a `sync.Mutex`.

**Q: Difference between a Queue and a Stack?**
**A:** Order of processing. Queue is FIFO (First-In, First-Out) like a line at a store. Stack is LIFO (Last-In, First-Out) like a stack of plates.
