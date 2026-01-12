---
---

## Summary
**Back Pressure** is a flow control mechanism used when a consumer cannot keep up with the rate of data produced by the producer. Instead of overwhelming the consumer (leading to crashes or memory leaks), the system signals the producer to slow down or drops excess load.

## Detailed Explanation

### Strategies
1.  **Buffering (Queueing)**: Store incoming requests in a queue.
    *   *Risk*: If the buffer fills up, you eventually run out of memory (OOM).
2.  **Dropping (Load Shedding)**:
    *   **Drop Newest**: Reject new requests immediately (HTTP 503). Preferred for real-time user requests.
    *   **Drop Oldest**: Discard old data to make room for new. Preferred for telemetry/metrics.
3.  **Flow Control (Throttling)**: The consumer explicitly tells the producer "Stop" or "Slow down" (e.g., TCP Windowing).

### Handling Overload
*   **Circuit Breaker**: Stop calling a failing service to give it time to recover.
*   **Rate Limiting**: Limit the number of requests a user can make per second.

## Go Context: Unbuffered Channels
In Go, an unbuffered channel naturally provides back pressure.

```go
ch := make(chan int) // Unbuffered

// Producer
go func() {
    // This BLOCKS until the consumer is ready to receive.
    // The producer cannot run faster than the consumer.
    ch <- 1 
}()

// Consumer
go func() {
    time.Sleep(1 * time.Second) // Slow consumer
    val := <-ch
}()
```

## Interview Questions

### Q: What happens if you don't implement back pressure?
**A:** The "Thundering Herd" or "Cascading Failure" problem. If a service slows down, requests pile up in memory. This consumes more RAM, causing Garbage Collection (GC) pauses, making the service even slower, until it crashes. The retry storm from clients then crashes the replicas.

### Q: How does TCP handle back pressure?
**A:** TCP uses a "Sliding Window" mechanism. The receiver advertises its "Window Size" (how much data it can buffer). If the window is full, the sender stops transmitting until the receiver acknowledges data.
