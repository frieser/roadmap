---
---

## Summary
**CPU Cache** is a small, extremely fast memory located directly on the CPU die. It stores copies of frequently used data from main memory (RAM) to speed up access. The hierarchy typically consists of **L1** (fastest/smallest), **L2**, and **L3** (slower/larger, shared).

## Detailed Explanation
### Memory Hierarchy Latency (Approx)
*   **L1 Cache**: ~1 ns
*   **L2 Cache**: ~4 ns
*   **L3 Cache**: ~10+ ns
*   **RAM**: ~100 ns

### Locality of Reference
Caches work because of two principles:
1.  **Temporal Locality**: If you access data now, you will likely access it again soon (e.g., loop counter).
2.  **Spatial Locality**: If you access address X, you will likely access X+1 soon (e.g., iterating an array).

### Cache Lines
Data is loaded into cache in chunks called **Cache Lines** (usually 64 bytes). Reading one byte actually loads 64 bytes.

### Go Context
*   **Cache-Friendly Code**: Go Slices (arrays) are contiguous in memory, making them very cache-friendly (high spatial locality). Linked Lists are scattered, causing frequent cache misses.
*   **False Sharing**: In concurrent Go programs, if two Goroutines write to independent variables that happen to sit on the *same cache line*, the cores fight over ownership of that line, killing performance.
    *   *Fix*: Add padding fields to structs to push variables into different cache lines.

```go
type PaddedStruct struct {
    A uint64
    _ [56]byte // Padding to fill 64-byte cache line
    B uint64
}
```

## Interview Questions
**Q: Why is iterating over a Slice faster than a Linked List?**
A: Spatial Locality. A slice's data is contiguous. The CPU loads a chunk (Cache Line) and gets the next few elements "for free". A Linked List's nodes are scattered in heap memory, causing a Cache Miss (RAM access) for almost every node.

**Q: What is a Cache Miss?**
A: When the CPU requests data that is not currently in the cache. It must verify fetch it from slower RAM (or L2/L3), stalling execution.

## Diagram
```mermaid
graph TD
    CPU --> L1[L1 Cache]
    L1 --> L2[L2 Cache]
    L2 --> L3[L3 Cache (Shared)]
    L3 --> RAM[Main Memory]
    
    style L1 fill:#f9f,stroke:#333
    style RAM fill:#999,stroke:#333
```
