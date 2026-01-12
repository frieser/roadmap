# pprof (Profiling)

## Summary
`pprof` is the standard tool for profiling Go programs. It helps you visualize and analyze where your program spends its CPU time and memory. It consists of two parts: the runtime libraries (`runtime/pprof`, `net/http/pprof`) to generate profiles, and the `go tool pprof` command-line tool to analyze them interactively or via web UI.

## Detailed Explanation

### 1. Enabling Profiling
For web services, the easiest way is to import `net/http/pprof`. This automatically registers handlers at `/debug/pprof/`.

```go
import _ "net/http/pprof"

func main() {
    // Starts a web server for profiling on port 6060
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    // ... app logic ...
}
```

### 2. Types of Profiles
*   **cpu**: Samples where the program spends execution time.
*   **heap**: Tracks memory allocations (current usage and total allocations).
*   **goroutine**: Stack traces of all current goroutines.
*   **block**: Where goroutines block waiting on synchronization primitives.
*   **mutex**: Contention on mutex locks.

### 3. Analyzing Profiles
Use the `go tool` to inspect the live application.
```bash
# Interactive CLI
go tool pprof http://localhost:6060/debug/pprof/heap

# Web UI (Flame Graph)
go tool pprof -http=:8081 http://localhost:6060/debug/pprof/profile?seconds=30
```

### 4. Flame Graphs
The Web UI provides a "Flame Graph" view, which is the most intuitive way to spot bottlenecks.
*   **Width**: Represents the percentage of resources (CPU time or RAM) consumed.
*   **Y-Axis**: Stack depth.
*   Look for "wide" bars at the bottom (functions consuming lots of resources) or wide "towers" (deep call stacks doing heavy work).

## Interview Questions

**Q: What is the overhead of enabling `net/http/pprof` in production?**
**A:** Simply importing the package and exposing the endpoint has **near-zero overhead** because profiling is not active until a request is made. However, *running* a CPU profile (e.g., for 30 seconds) does add a small CPU overhead (usually < 5%) and pauses the garbage collector briefly for some snapshots. It is generally safe to leave the endpoints enabled in production, provided they are protected (not publicly accessible) to prevent DoS attacks or information leakage.

**Q: How do you identify a memory leak using pprof?**
**A:** You compare two heap profiles taken at different times (e.g., "base" and "diff").
```bash
go tool pprof -diff_base=base.heap current.heap
```
Or simply look at the `inuse_space` metric in the heap profile. If the `inuse_space` for a specific function keeps growing over time without being released, that's a leak candidate.

**Q: What is the difference between `flat` and `cum` (cumulative) numbers in pprof output?**
**A:**
*   **Flat**: How much time/memory was spent *directly* inside this function (excluding calls to other functions).
*   **Cum**: How much time/memory was spent in this function *and* all functions it called (its children).
A function with high `flat` is a hotspot itself. A function with high `cum` but low `flat` is an orchestrator calling expensive children.
