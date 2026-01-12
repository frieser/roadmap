#API
---
---

## Summary
Rate limiting is a strategy for limiting network traffic. It puts a cap on how often someone can repeat an action within a certain timeframe – for instance, trying to log in to an account. Rate limiting is essential for protecting APIs from malicious attacks like DDoS, brute force, and ensuring fair usage among consumers by preventing any single client from exhausting system resources.

## Detailed Explanation

### Why Rate Limit?
1.  **Preventing DoS/DDoS Attacks**: By limiting the number of requests a client can make, you can mitigate the impact of flooding attacks that aim to take down your service.
2.  **Resource Management**: APIs often rely on backend resources (databases, memory, CPU). Rate limiting prevents these resources from being overwhelmed.
3.  **Cost Control**: For pay-per-use infrastructure (like serverless or third-party APIs), rate limiting helps manage and predict costs.
4.  **Fairness**: Ensures that no single "noisy neighbor" can hog the bandwidth or server capacity, providing a consistent experience for all users.
5.  **Security**: Protects against brute-force attacks on sensitive endpoints like \`/login\` or \`/password-reset\`.

### Rate Limiting Algorithms

#### 1. Token Bucket
*   **Concept**: A "bucket" holds tokens. Tokens are added at a fixed rate. Each request removes a token. If the bucket is empty, the request is rejected.
*   **Pros**: Supports bursts of traffic (up to the bucket size). Very memory efficient.
*   **Go Implementation**: This is the algorithm used by \`golang.org/x/time/rate\`.

#### 2. Leaky Bucket
*   **Concept**: Similar to a bucket with a hole at the bottom. Requests enter the bucket at any rate but leak (are processed) at a fixed, constant rate.
*   **Pros**: Smooths out traffic spikes. Provides a stable outflow rate.

#### 3. Fixed Window Counter
*   **Concept**: Divides time into fixed intervals (e.g., 1-minute windows). A counter tracks requests in each window.
*   **Cons**: "Boundary problem" – a client can double the limit by sending requests at the end of one window and the start of the next.

#### 4. Sliding Window Log
*   **Concept**: Records a timestamp for every request. To check the limit, it counts logs within the last \$T\$ duration.
*   **Pros**: Extremely accurate.
*   **Cons**: High memory overhead because it stores every request's timestamp.

#### 5. Sliding Window Counter
*   **Concept**: A hybrid approach. It uses the count from the current fixed window and the previous window, weighting them based on how much time has passed in the current window.
*   **Formula**: \`count = current_window_count + previous_window_count * (remaining_time_in_previous_window / window_duration)\`

### Rate Limiting Headers
When an API is rate-limited, it should communicate the status to the client using standard HTTP headers:
*   \`X-RateLimit-Limit\`: The maximum number of requests allowed in a period.
*   \`X-RateLimit-Remaining\`: The number of requests remaining in the current window.
*   \`X-RateLimit-Reset\`: The time at which the current rate limit window resets (Epoch time).
*   **Status Code 429**: The standard HTTP status code for "Too Many Requests".
*   \`Retry-After\`: Often returned with a 429 to tell the client how many seconds to wait before retrying.

### Implementation in Go
Go's standard sub-package \`golang.org/x/time/rate\` provides an efficient Token Bucket implementation.

\`\`\`go
package main

import (
	"context"
	"fmt"
	"net/http"
	"time"

	"golang.org/x/time/rate"
)

// RateLimiter wraps a rate.Limiter
type RateLimiter struct {
	limiter *rate.Limiter
}

func NewRateLimiter(r rate.Limit, b int) *RateLimiter {
	return &RateLimiter{
		limiter: rate.NewLimiter(r, b),
	}
}

// Middleware handles the rate limiting logic
func (rl *RateLimiter) Middleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Allow() returns true if the event is allowed to happen
		if !rl.limiter.Allow() {
			http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
			return
		}
		next.ServeHTTP(w, r)
	})
}

func main() {
	// Create a limiter that allows 5 requests per second with a burst of 10
	limiter := NewRateLimiter(5, 10)

	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Hello, limited world!")
	})

	server := &http.Server{
		Addr:    ":8080",
		Handler: limiter.Middleware(mux),
	}

	fmt.Println("Server starting on :8080...")
	server.ListenAndServe()
}
\`\`\`

## Interview Questions

**Q: What is the difference between Throttling and Rate Limiting?**
**A:** Rate limiting is a hard cap on requests (e.g., 100 req/min). Throttling is a broader term that often involves slowing down requests (increasing latency) rather than outright rejecting them, although they are often used interchangeably in API design.

**Q: How do you handle rate limiting in a distributed system with multiple API instances?**
**A:** Local rate limiting (in-memory) won't work because different instances don't share state. You typically use a centralized fast-access store like **Redis** to keep track of counters across all instances.

**Q: Why is the Token Bucket algorithm preferred over Fixed Window?**
**A:** Fixed Window suffers from boundary bursts, where a user can send twice the limit in a very short time if they time it around the window reset. Token Bucket supports bursts gracefully and refills smoothly.

**Q: What should a client do when it receives a 429 Too Many Requests response?**
**A:** The client should stop sending requests immediately and wait for the duration specified in the \`Retry-After\` header. Implementing **Exponential Backoff** with jitter is a best practice for retrying.
