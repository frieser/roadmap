#API
---
---

# API Rate Limiting and Throttling

## Summary
Rate Limiting controls the **rate** of traffic sent or received by a network interface controller. In API design, it is a defensive mechanism used to prevent **Denial of Service (DoS)** attacks, ensure fair usage among tenants (in SaaS), and protect backend resources from becoming overwhelmed.

*   **Rate Limiting**: Capping the number of requests a user can make in a given timeframe (e.g., 100 req/min). Exceeding this results in a `429 Too Many Requests` error.
*   **Throttling**: Controlling the bandwidth or processing speed. Instead of rejecting requests immediately, the server might process them slower (queueing) or degrade the service quality.

## Detailed Explanation

### Algorithms
1.  **Token Bucket**: A bucket is filled with tokens at a constant rate. Each request consumes a token. If the bucket is empty, the request is rejected. Allows for "bursts" of traffic up to the bucket size.
2.  **Leaky Bucket**: Requests enter a bucket and "leak" out (are processed) at a constant rate. Useful for smoothing out traffic bursts into a steady stream.
3.  **Fixed Window**: A counter is reset every minute. Can allow 2x the limit at the edges of the window (e.g., 100 reqs at 10:59:59 and 100 reqs at 11:00:01).
4.  **Sliding Window**: A more accurate approximation that solves the "boundary double-limit" issue of Fixed Windows by weighing the request count of the previous window.

### Token Bucket Visualization
```mermaid
graph TD
    Source[Token Refill Source] -->|Adds Tokens (Rate r)| Bucket{Bucket}
    Bucket -->|Capacity b| Overflow[Overflow Discarded]
    User[API Request] -->|Needs 1 Token| Check{Has Token?}
    Check -->|Yes| Consume[Remove Token]
    Consume --> Process[Process Request (200 OK)]
    Check -->|No| Reject[Reject Request (429 Too Many Requests)]
```

### Implementation Strategies
*   **In-Memory (Single Instance)**: Fast, but doesn't scale. If you have 5 API servers, the limit is 5x what you intended, and state isn't shared.
*   **Distributed (Redis)**: Uses a central store (like Redis) to count requests across all API instances. Often uses Lua scripts to ensure atomicity of "check-and-decrement" operations.

### Go Implementation
Go's standard library provides `golang.org/x/time/rate` which implements the **Token Bucket** algorithm.

#### 1. Simple Middleware with `golang.org/x/time/rate`
This example creates a rate limiter allowing 1 request per second with a burst of 5.

```go
package main

import (
	"net/http"
	"golang.org/x/time/rate"
)

var limiter = rate.NewLimiter(1, 5) // 1 event/sec, burst of 5

func limitMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		if !limiter.Allow() {
			http.Error(w, "429 Too Many Requests", http.StatusTooManyRequests)
			return
		}
		next.ServeHTTP(w, r)
	})
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello, World!"))
	})

	http.ListenAndServe(":8080", limitMiddleware(mux))
}
```

#### 2. Advanced: Per-IP Limiting with `tollbooth`
In production, you limit *per user* or *per IP*. The `tollbooth` library simplifies this.

```go
package main

import (
    "net/http"
    "github.com/didip/tollbooth/v7"
)

func main() {
    // Create a limiter that allows 1 request per second
    lmt := tollbooth.NewLimiter(1, nil)

    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, World!"))
    })

    // Wrap the handler
    http.ListenAndServe(":8080", tollbooth.LimitFuncHandler(lmt, handler))
}
```

## Interview Questions

1.  **What happens when a Token Bucket is empty vs. a Leaky Bucket is full?**
    *   *Answer:* In a Token Bucket, if it's empty, new requests are rejected immediately (or queued). In a Leaky Bucket, if it's full, new requests are discarded (overflow). Token Bucket allows bursts; Leaky Bucket enforces a constant output rate.

2.  **How do you handle Rate Limiting in a distributed system with multiple Go binaries?**
    *   *Answer:* You cannot use in-memory limiters (like `x/time/rate`) effectively. You must use a centralized store like **Redis**. A common pattern is the "Fixed Window" or "Sliding Window Log" using Redis atomic counters (`INCR`, `EXPIRE`) or Lua scripts to synchronize limits across all instances.

3.  **Why might you prefer "Throttling" (slowing down) over hard "Rate Limiting" (429 errors)?**
    *   *Answer:* Throttling provides a better user experience for slight excesses. Instead of breaking the client application with an error, the response is just delayed. This is common in background processing APIs where latency is less critical than reliability.

4.  **What headers should you send with a 429 response?**
    *   *Answer:* `X-RateLimit-Limit` (The ceiling), `X-RateLimit-Remaining` (Requests left in window), and `X-RateLimit-Reset` (Time/Seconds until the window resets). `Retry-After` is also crucial to tell the client when to come back.

5.  **What is the "Thundering Herd" problem in the context of rate limiting?**
    *   *Answer:* If many clients hit the rate limit and all retry at the exact same time (when the window resets), it causes a massive spike. To mitigate this, clients should use **Exponential Backoff** with **Jitter** (randomness) in their retry logic.