# Garbage Collector

## Summary
Go uses a **non-generational, concurrent, tricolor mark-and-sweep** garbage collector. It is designed for low latency (sub-millisecond pauses) rather than maximum throughput. It runs concurrently with the application code, utilizing **Write Barriers** to maintain consistency during the marking phase.

## Detailed Explanation

### The Tricolor Algorithm
The GC classifies heap objects into three colors:
1.  **White**: Potential garbage. Not yet visited. At the end of the scan, white objects are reclaimed.
2.  **Grey**: Visited, but its children (references) have not been scanned yet.
3.  **Black**: Visited and all children scanned. Safe from collection.

### GC Phases
1.  **Mark Setup (STW)**: A very short Stop-The-World pause to enable Write Barriers.
2.  **Marking (Concurrent)**: The GC scans the heap, turning White objects Grey, then Black. It uses ~25% of CPU capacity.
    *   **Write Barriers**: If the application modifies a pointer while GC is running (e.g., Black -> White), the barrier catches this and colors the object Grey to prevent accidental collection.
3.  **Mark Termination (STW)**: A short pause to finish pending tasks and disable Write Barriers.
4.  **Sweep (Concurrent)**: The allocator reclaims memory from White objects as needed (lazy sweeping).

### Tuning: GOGC
The primary knob for tuning is the `GOGC` environment variable (default: 100).
*   **GOGC=100**: Trigger GC when the heap grows by 100% (doubles) since the last collection.
*   **GOGC=200**: Trigger when heap grows by 200% (triples). Reduces GC frequency but uses more RAM.
*   **GOGC=off**: Disables GC entirely.

### Code Example: Forcing GC and Monitoring
Usually, you let the runtime handle it, but you can force it or read stats.

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    // Print GC stats
    var m runtime.MemStats
    
    // Allocate some memory
    s := make([]int, 0, 10000)
    for i := 0; i < 10000; i++ {
        s = append(s, i)
    }

    // Force GC (Blocking)
    fmt.Println("Forcing GC...")
    runtime.GC()

    // Read stats
    runtime.ReadMemStats(&m)
    fmt.Printf("Heap Alloc: %v bytes\n", m.HeapAlloc)
    fmt.Printf("NumGC: %v\n", m.NumGC)
}
```

## Interview Questions

**Q: Is Go's GC generational?**
**A:** No. It is a mark-and-sweep collector. It does not move objects (non-compacting) and does not separate young/old generations, which keeps it simple and avoids write barriers for every pointer read/write (though it uses them for the concurrent mark phase).

**Q: What is a Write Barrier?**
**A:** A mechanism that ensures data consistency during the concurrent marking phase. If the application code modifies a pointer while the GC is scanning, the write barrier alerts the GC (colors the object grey) so it doesn't mistakenly collect a live object.

**Q: How do you tune the Go GC?**
**A:** Primarily via the `GOGC` environment variable. Increasing it reduces GC frequency at the cost of higher memory usage.
