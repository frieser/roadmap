# Benchmarks

## Summary
Go has built-in support for benchmarking via the `testing` package. Benchmarks allow you to measure the performance of your code (execution time and memory allocation) to identify bottlenecks. They are defined in `_test.go` files with functions starting with `Benchmark` and use the `testing.B` type to control the loop.

## Detailed Explanation

### Writing a Benchmark
A benchmark function looks like a test but takes `*testing.B` and must contain a loop that runs `b.N` times.

```go
func BenchmarkName(b *testing.B) {
    // Optional setup
    for i := 0; i < b.N; i++ {
        // Code to measure
    }
}
```

*   **`b.N`**: The testing framework dynamically adjusts this number (1, 100, 10000...) until the benchmark runs for a sufficient time (default 1 second) to get a stable measurement.

### Code Example

**`concat_test.go`**
```go
package main

import (
    "fmt"
    "testing"
)

func BenchmarkSprintf(b *testing.B) {
    // Reset timer to ignore setup costs if any
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        _ = fmt.Sprintf("hello %s", "world")
    }
}
```

### Running Benchmarks
*   `go test -bench .`: Run all benchmarks in current directory.
*   `go test -bench . -benchmem`: Include memory allocation statistics (B/op, allocs/op).
*   `go test -bench BenchmarkName -run ^$`: Run specific benchmark and skip all tests (using regex `^$` to match no test names).

### Interpreting Output
```
BenchmarkSprintf-8    20000000    85.2 ns/op    16 B/op    1 allocs/op
```
*   **Suffix (-8)**: Number of GOMAXPROCS (CPUs) used.
*   **20000000**: Number of iterations (`b.N`) run.
*   **85.2 ns/op**: Average time per operation (nanoseconds).
*   **16 B/op**: Bytes allocated per operation.
*   **1 allocs/op**: Number of heap allocations per operation.

### Common `testing.B` Methods
*   `b.ResetTimer()`: Reset the clock. Use this after expensive setup code that shouldn't be measured.
*   `b.StopTimer()` / `b.StartTimer()`: Pause/resume timing (e.g., to prepare data inside the loop).
*   `b.ReportAllocs()`: Equivalent to running with `-benchmem`.

## Interview Questions

**Q: Why is the `for i := 0; i < b.N; i++` loop required in a benchmark?**
**A:** The Go testing framework determines the value of `b.N` dynamically. It starts with small values and increases them until the benchmark runs for a statistically significant amount of time (usually 1 second). The loop ensures the code under test is executed exactly the number of times requested by the harness to calculate accurate averages.

**Q: What does `b.ResetTimer()` do and when should you use it?**
**A:** It resets the elapsed time and memory allocation counters to zero. You should use it if you have expensive setup logic (like generating a large dataset) *before* the loop that you don't want to include in the performance metric of the function being benchmarked.

**Q: How do you benchmark a function that takes arguments?**
**A:** You cannot pass arguments directly to a benchmark function. Instead, you create a helper function or an anonymous function wrapper.
```go
func benchmarkFib(i int, b *testing.B) {
    for n := 0; n < b.N; n++ {
        Fib(i)
    }
}

func BenchmarkFib10(b *testing.B) { benchmarkFib(10, b) }
func BenchmarkFib20(b *testing.B) { benchmarkFib(20, b) }
```
