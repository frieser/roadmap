# Go Trace

## Summary
`go tool trace` provides a fine-grained, nanosecond-level visualization of the Go runtime execution. Unlike `pprof` (which aggregates samples), `trace` captures **events**: goroutine scheduling, garbage collection (GC) pauses, network blocking, and system calls. It is the ultimate tool for debugging latency issues and understanding concurrency behavior.

## Detailed Explanation

### How to Collect a Trace
1.  **Code**: Use `runtime/trace`.
    ```go
    f, _ := os.Create("trace.out")
    trace.Start(f)
    defer trace.Stop()
    ```
2.  **Test**: Use the flag.
    ```bash
    go test -trace=trace.out
    ```

### Visualization
Run `go tool trace trace.out`. This opens a Chrome/Edge browser window with a complex UI:
1.  **View trace**: A timeline showing every logical processor (P) and what goroutine (G) was running on it at any microsecond.
2.  **Goroutine analysis**: Shows how long each goroutine spent in different states (Running, Runnable, Waiting, Syscall).
3.  **Network blocking**: Shows when network I/O blocked execution.

### When to use Trace vs Pprof
*   **Use Pprof** when: The CPU usage is high, or memory usage is high. You want to know "which function is slow?".
*   **Use Trace** when: The CPU usage is **low**, but latency is **high**. You want to know "why is my program sleeping?" or "is the GC pausing my app too often?".

## Interview Questions

**Q: What can `go tool trace` reveal that `pprof` cannot?**
**A:** `trace` reveals **latency** causes that don't consume CPU, such as:
*   Goroutines waiting for a mutex.
*   Goroutines waiting for network I/O.
*   Poor scheduling (e.g., many goroutines fighting for few processors).
*   GC "Stop-The-World" pause latencies.
`pprof` cpu profiles only show you what code is *running*, not what code is *waiting*.

**Q: What does the "Runnable" state mean in the trace viewer?**
**A:** It means the goroutine is ready to execute (not blocked on I/O or mutex) but is **waiting for a CPU thread** to become available. If you see significant time spent in the "Runnable" state, it indicates CPU saturation or scheduler contention—you might have too many active goroutines for the available GOMAXPROCS.

**Q: How does the trace tool help debug Garbage Collection issues?**
**A:** The trace timeline clearly visualizes GC phases (Marking, Sweeping) and "Stop-The-World" (STW) pauses. You can see exactly when the application execution stops entirely for GC and how long those pauses last, allowing you to correlate latency spikes with GC activity.
