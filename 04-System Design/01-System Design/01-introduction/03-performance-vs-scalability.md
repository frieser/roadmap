---
---

## Summary
**Performance** is about "speed" (how fast a single unit of work is done), while **Scalability** is about "growth" (how well the system handles increasing load). A system can be performant but not scalable (fast for 1 user, crashes with 100), or scalable but not performant (handles 1M users, but everyone waits 2 seconds).

## Detailed Explanation

### Performance (Speed)
*   **Definition**: The time it takes to process a single request.
*   **Key Metric**: **Latency** (Response time).
*   **Optimization**: Algorithms, code profiling, caching, concurrency, database indexing.
*   **Go Context**: Using **Goroutines** to maximize CPU usage and handle concurrent I/O efficiently on a single machine.

### Scalability (Capacity)
*   **Definition**: The ability of the system to cope with increased load by adding resources.
*   **Key Metric**: **Throughput** (Requests Per Second - RPS).
*   **Optimization**: Distributed systems, partitioning, load balancing, stateless services.
*   **Go Context**: Building **Microservices** that can be replicated across hundreds of nodes (Horizontal Scaling).

### Vertical vs. Horizontal Scaling

| Feature | Vertical (Scale Up) | Horizontal (Scale Out) |
| :--- | :--- | :--- |
| **Action** | Add more CPU/RAM to one server | Add more servers to the cluster |
| **Limit** | Hardware ceiling | Theoretically infinite |
| **Complexity** | Low (no code changes) | High (requires distributed logic) |
| **Cost** | Exponential (high-end hardware) | Linear (commodity hardware) |
| **Failover** | Single Point of Failure | High Redundancy |

## Go Example

### Optimizing Performance (Concurrency)
Using a buffered channel and worker pool to process items faster on one node.

```go
func worker(jobs <-chan int, results chan<- int) {
    for n := range jobs {
        results <- expensiveCalculation(n)
    }
}
```

### Optimizing Scalability (Statelessness)
Designing a handler that doesn't store session state locally, allowing it to run on any replica.

```go
func Handler(w http.ResponseWriter, r *http.Request) {
    // State is stored in external Redis, not in memory
    session, _ := redisClient.Get(r.Context(), "session_id").Result()
    // ...
}
```

## Interview Questions

### Q: Can a system be scalable but not performant?
**A:** Yes. Imagine a distributed system with 100 microservices. It can scale to handle millions of requests by adding more servers, but if a single request has to hop through 20 services, the latency (performance) might be high (e.g., 500ms).

### Q: When should you choose Vertical Scaling over Horizontal?
**A:** When the load is predictable, the complexity of a distributed system isn't justified, or for database layers where sharding/clustering is operationally expensive. It's often the best "first step" before re-architecting for horizontal scale.
