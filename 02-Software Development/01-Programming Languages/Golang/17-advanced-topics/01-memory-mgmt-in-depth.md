# Memory Management In-Depth

## Summary
Go's memory management is automatic, handling allocation and deallocation so developers don't have to. It uses a **TCMalloc-inspired allocator** designed for low fragmentation and high concurrency. The runtime divides memory into **Stack** (local variables, fast) and **Heap** (shared variables, managed by GC). The heap allocator uses a hierarchy of structures (`mcache`, `mcentral`, `mheap`) to minimize locking and optimize for different object sizes.

## Detailed Explanation

### Stack vs Heap
*   **Stack**: Memory attached to a goroutine. Allocation is extremely fast (pointer bump). Used for local variables, parameters, and return values that do not escape. Stacks grow and shrink dynamically (starting at 2KB).
*   **Heap**: Global memory pool for dynamic data that outlives a function call or is shared between goroutines. Managed by the Garbage Collector. Allocation is slower and incurs GC overhead.

### The Allocator Hierarchy
Go's allocator breaks memory down into pages and spans to reduce lock contention:

1.  **mcache**: Per-P (Processor) cache. Stores a local cache of spans for small objects. **No locks required** for allocation here, making it very fast.
2.  **mcentral**: Central list of spans. When `mcache` is empty, it requests a new span from `mcentral`. Requires locking.
3.  **mheap**: The global heap. Manages large objects (>32KB) and pages of memory. Requests memory from the OS when needed.

### Object Size Classes
*   **Tiny (< 16B)**: Allocated in a single block (16B) within the `mcache` to reduce fragmentation.
*   **Small (16B - 32KB)**: Allocated from the corresponding size class in `mcache`.
*   **Large (> 32KB)**: Allocated directly from `mheap`.

### Code Example: Stack vs Heap
The compiler decides where data lives via **Escape Analysis**.

```go
package main

import "fmt"

type User struct {
    ID   int
    Name string
}

// stayedOnStack creates a user that is never passed outside.
// The compiler allocates 'u' on the stack.
func stayedOnStack() {
    u := User{ID: 1, Name: "Stack"}
    _ = u.ID
}

// escapedToHeap returns a pointer, so the data must survive function exit.
// The compiler allocates 'u' on the heap.
func escapedToHeap() *User {
    u := User{ID: 2, Name: "Heap"}
    return &u
}

func main() {
    stayedOnStack()
    u := escapedToHeap()
    fmt.Println(u.Name)
}
```

You can verify this with:
```bash
go build -gcflags '-m' main.go
# Output:
# ./main.go:19:9: &u escapes to heap
# ./main.go:18:2: moved to heap: u
```

## Interview Questions

**Q: Explain the hierarchy of Go's memory allocator.**
**A:** It consists of `mcache` (per-thread, no locks, for small objects), `mcentral` (shared list of spans, requires locks), and `mheap` (global manager for large objects and OS requests).

**Q: What is the difference between stack and heap allocation in Go?**
**A:** Stack allocation is for local variables and is cheap (CPU instruction). Heap allocation is for data that escapes the function, requires the GC to manage, and is more expensive.

**Q: Why does Go use a TCMalloc-style allocator?**
**A:** To minimize fragmentation and lock contention in highly concurrent applications. The per-processor `mcache` allows most allocations to happen without locks.
