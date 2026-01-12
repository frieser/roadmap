---
---

## Summary
Throttling (often used interchangeably with Rate Limiting, though subtly different) is a mechanism to control the rate at which requests are processed by a server. It protects resources from being overwhelmed by too many requests from a single user, IP, or service, ensuring stability and fair usage.

## Detailed Explanation

### Throttling vs Rate Limiting
*   **Rate Limiting**: "You can make 100 requests per hour." (Quota focus). Rejects requests over the limit.
*   **Throttling**: "You can make requests, but I will only process 1 per second." (Speed/Congestion focus). Often queues or delays requests to smooth out spikes.
In practice, backend engineers often use the terms to mean "limiting traffic."

### Algorithms
1.  **Token Bucket**: Tokens are added to a bucket at a fixed rate. Each request costs a token. If the bucket is empty, the request is denied. Allows for "bursts" (up to bucket size).
2.  **Leaky Bucket**: Requests enter a queue (bucket) and are processed (leaked) at a constant rate. Smooths out traffic but has a fixed queue size.
3.  **Fixed Window**: "100 reqs per minute". Resets at the top of the minute. Can allow 200 reqs in 1 sec (end of minute 1 + start of minute 2).

## Go-Specific Context/Examples

Go's standard library `golang.org/x/time/rate` implements the **Token Bucket** algorithm.

### Example: Simple Rate Limiter Middleware

```go
package main

import (
	"fmt"
	"net/http"
	"time"

	"golang.org/x/time/rate"
)

// Allow 1 request per second, with a burst of 3
var limiter = rate.NewLimiter(1, 3)

func limitMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Attempt to consume a token
		if !limiter.Allow() {
			http.Error(w, "Too Many Requests", http.StatusTooManyRequests) // 429
			return
		}
		next.ServeHTTP(w, r)
	})
}

func mainHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Request allowed at %v", time.Now().Format(time.TimeOnly))
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", mainHandler)

	fmt.Println("Server with rate limiting started on :8080")
	// Try refreshing the page quickly to see 429 errors
	http.ListenAndServe(":8080", limitMiddleware(mux))
}
```

### Distributed Throttling
For distributed systems (multiple server instances), in-memory Go rate limiters won't work (limits would apply per server, not globally). You typically use an external store like **Redis** (e.g., incrementing a key `user:123:req_count` with an expiry) to maintain global counts.

## Interview Questions

**Q: What HTTP status code should you return when throttling a user?**
**A:** `429 Too Many Requests`. It is also best practice to include a `Retry-After` header indicating how many seconds the client should wait before trying again.

**Q: Explain the "Thundering Herd" problem and how throttling helps.**
**A:** Thundering Herd occurs when a large number of processes/users wake up or retry simultaneously (e.g., after a service recovers), causing an immediate spike that crashes the service again. Throttling/Rate Limiting prevents this by capping the maximum load, and adding "Jitter" (randomness) to retry intervals helps spread the load.

**Q: What is the difference between Token Bucket and Leaky Bucket?**
**A:** Token Bucket allows for **bursts** of traffic (you can use all accumulated tokens at once). Leaky Bucket enforces a **constant flow** rate (smoothing out traffic), regardless of how bursty the input is. Token Bucket is generally preferred for user APIs to allow better UX (short bursts of activity).
