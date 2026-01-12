---
---

# Multi-threaded

## Summary
**Multi-threaded** problem solving involves utilizing concurrency to perform tasks in parallel. In Go, this is handled natively via **Goroutines** and **Channels**.

## Detailed Explanation

### Core Concepts
*   **Goroutine**: Lightweight thread managed by Go runtime.
*   **Channel**: Typed conduit for communication and synchronization.
*   **WaitGroup**: Wait for a collection of goroutines to finish.
*   **Mutex**: Protect shared memory (Critical Section).

## Code Examples (Go)

### Worker Pool Pattern
Distribute jobs to workers.

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        results <- j * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)
    
    // Start 3 workers
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }
    
    // Send jobs
    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs)
    
    // Collect results
    for a := 1; a <= 5; a++ {
        <-results
    }
}
```
