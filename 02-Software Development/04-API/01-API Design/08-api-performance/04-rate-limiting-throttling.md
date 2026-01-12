# Rate Limiting & Throttling

## Summary
Rate Limiting controls the rate of traffic sent or received by a network interface controller. It is essential for protecting APIs from abuse (DDoS), ensuring fair usage among users (quotas), and preventing cascading failures due to overload. Common algorithms include **Token Bucket** and **Leaky Bucket**. When limits are exceeded, the API should return `429 Too Many Requests`.

## Detailed Explanation

### Concepts
*   **Rate Limiting**: Capping the number of requests a user can make in a specific time window (e.g., 100 req/min). Used for business quotas and security.
*   **Throttling**: Intentionally slowing down a service to regulate network traffic. Often implies a temporary degradation rather than a hard stop.

### Algorithms
1.  **Token Bucket**:
    *   Tokens are added to a bucket at a fixed rate.
    *   Each request consumes a token.
    *   If bucket is empty, request is rejected.
    *   *Pros*: Allows bursts of traffic.
    *   *Implementation*: `golang.org/x/time/rate`.

2.  **Leaky Bucket**:
    *   Requests enter a bucket (queue) and are processed at a constant rate.
    *   If bucket overflows, requests are discarded.
    *   *Pros*: Smooths out traffic (constant outflow).

3.  **Fixed Window Counter**:
    *   Counts requests per time unit (e.g., 10:00-10:01).
    *   *Cons*: Spike at window edges (e.g., 100 reqs at 10:00:59 and 100 at 10:01:01 allows double rate).

4.  **Sliding Window Log**:
    *   Tracks timestamp of each request. Most accurate but expensive (storage).

### HTTP Headers
Standard headers to communicate limits:
*   `X-RateLimit-Limit`: The ceiling for the time window.
*   `X-RateLimit-Remaining`: Requests left in current window.
*   `X-RateLimit-Reset`: Time when window resets (Unix timestamp).
*   `Retry-After`: Seconds to wait before retrying (sent with 429).

### Go Implementation: Token Bucket (Per IP)
Using `golang.org/x/time/rate` and a map for per-IP limiting.

```go
package main

import (
	"net"
	"net/http"
	"sync"
	"time"

	"golang.org/x/time/rate"
)

// IPRateLimiter holds rate limiters for each visitor
type IPRateLimiter struct {
	ips map[string]*rate.Limiter
	mu  sync.Mutex
	r   rate.Limit
	b   int
}

func NewIPRateLimiter(r rate.Limit, b int) *IPRateLimiter {
	i := &IPRateLimiter{
		ips: make(map[string]*rate.Limiter),
		r:   r,
		b:   b,
	}
	// Background cleanup routine omitted for brevity
	return i
}

func (i *IPRateLimiter) GetLimiter(ip string) *rate.Limiter {
	i.mu.Lock()
	defer i.mu.Unlock()

	limiter, exists := i.ips[ip]
	if !exists {
		limiter = rate.NewLimiter(i.r, i.b)
		i.ips[ip] = limiter
	}

	return limiter
}

func limitMiddleware(limiter *IPRateLimiter, next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		ip, _, _ := net.SplitHostPort(r.RemoteAddr)
		
		lim := limiter.GetLimiter(ip)
		if !lim.Allow() {
			http.Error(w, "429 Too Many Requests", http.StatusTooManyRequests)
			return
		}

		next.ServeHTTP(w, r)
	})
}

func main() {
	// 1 request per second, burst of 3
	limiter := NewIPRateLimiter(1, 3)

	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Welcome!"))
	})

	http.ListenAndServe(":8080", limitMiddleware(limiter, mux))
}
```

## Interview Questions

**Q: Token Bucket vs Leaky Bucket?**
**A:** Token Bucket allows **bursts** of traffic up to the bucket capacity. Leaky Bucket enforces a **constant output rate**, smoothing out bursts. Use Token Bucket if you want to allow users to speed up briefly; use Leaky Bucket if your backend has strict processing limits (e.g., writing to a slow disk).

**Q: How do you handle distributed rate limiting?**
**A:** In a distributed system (multiple API instances), local memory limiters won't work globally. You must use a shared store like **Redis**. A common pattern is using Redis atomic counters with expiration (Fixed Window) or Lua scripts to implement Token Bucket atomically across nodes.

**Q: What is the downside of Fixed Window limiting?**
**A:** It suffers from the "boundary" or "stampede" effect. If a user makes their full quota of requests at the very end of one window and immediately again at the start of the next, they can effectively double their allowed rate for a short period, potentially overloading the system.
