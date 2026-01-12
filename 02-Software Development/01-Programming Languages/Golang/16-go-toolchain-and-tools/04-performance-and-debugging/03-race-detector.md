# Race Detector

## Summary
The Race Detector is a powerful tool built into the Go toolchain to find **data races**. A data race occurs when two goroutines access the same variable concurrently, and at least one of the accesses is a write. This leads to undefined behavior, crashes, and memory corruption. The detector is enabled via the `-race` flag.

## Detailed Explanation

### How it works
The race detector is based on the C/C++ ThreadSanitizer (TSan). When you compile with `-race`, the compiler injects code at every memory access (read/write) to record who accessed what and when. The runtime then monitors these access histories to detect unsynchronized concurrent access.

### Usage
```bash
go test -race ./...
go run -race main.go
go build -race -o myapp
```

### Limitations
1.  **Performance**: It increases memory usage by 5-10x and execution time by 2-20x.
2.  **No False Positives**: If it reports a race, it is a real race.
3.  **False Negatives**: It can only detect races that *actually happen* during execution. If a specific code path isn't triggered during your test, the race won't be found.

### Output
When a race is detected, the program prints a report to stderr and exits with code 66.
```
WARNING: DATA RACE
Write at 0x00c0000... by goroutine 7:
  main.main.func1() /path/to/main.go:12

Previous read at 0x00c0000... by goroutine 6:
  main.main.func2() /path/to/main.go:16
```

## Interview Questions

**Q: Can you run the race detector in production?**
**A:** Generally, **no**. The performance overhead (CPU and Memory) is too high for most production systems. However, it is occasionally done in "canary" deployments or staging environments under load to catch complex races that unit tests miss.

**Q: Does the race detector find deadlocks?**
**A:** No. The race detector finds **data races** (unsynchronized memory access). The Go runtime has a separate, built-in deadlock detector (fatal error: all goroutines are asleep), but that only catches situations where *all* goroutines are blocked. Partial deadlocks or logic bugs are not caught by either.

**Q: How do you fix a data race reported by the detector?**
**A:** You must synchronize access to the shared variable. Common solutions:
1.  Use `sync.Mutex` or `sync.RWMutex` to lock around the access.
2.  Use `atomic` package functions (e.g., `atomic.AddInt64`) for simple counters.
3.  Use Channels to communicate data instead of sharing memory ("Do not communicate by sharing memory; share memory by communicating").
