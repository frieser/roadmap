# Error Handling & Retries

## Summary
Robust API error handling ensures that failures are communicated clearly to clients, while retry mechanisms improve resilience against transient network issues. Key concepts include **Standard HTTP Status Codes**, **RFC 7807 (Problem Details)**, **Exponential Backoff**, **Jitter**, and the **Circuit Breaker** pattern to prevent cascading failures.

## Detailed Explanation

### Error Handling Best Practices
1.  **Use Standard HTTP Codes**:
    *   `2xx`: Success.
    *   `4xx`: Client Error (Bad Request, Unauthorized). Do NOT retry blindly.
    *   `5xx`: Server Error (Internal Error, Gateway Timeout). Safe to retry (sometimes).
2.  **Structured Error Responses**:
    *   Don't just send text. Send JSON.
    *   Follow **RFC 7807** (Problem Details for HTTP APIs).
    *   `{"type": "...", "title": "...", "status": 404, "detail": "..."}`.

### Retry Strategies
Retrying immediately can flood a struggling server (Thundering Herd).
1.  **Exponential Backoff**: Wait 1s, then 2s, then 4s, then 8s.
2.  **Jitter**: Add randomness to the wait time (e.g., wait 2s +/- 100ms) to prevent all clients from retrying at the exact same moment.

### Circuit Breaker Pattern
If a downstream service is down, retrying only adds load. A Circuit Breaker detects failures and "opens" the circuit, failing fast without calling the remote service, giving it time to recover.

### Go Implementation: Exponential Backoff with Jitter
Using a custom loop (or `github.com/cenkalti/backoff` for production).

```go
package main

import (
	"errors"
	"fmt"
	"math/rand"
	"time"
)

func flakyService() error {
	if rand.Float32() < 0.8 {
		return errors.New("temporary failure")
	}
	return nil
}

func retryWithBackoff(attempts int, sleep time.Duration) error {
	for i := 0; i < attempts; i++ {
		err := flakyService()
		if err == nil {
			fmt.Println("Success!")
			return nil
		}

		fmt.Printf("Attempt %d failed: %v. Retrying in %v...\n", i+1, err, sleep)

		// Add Jitter: +/- 10%
		jitter := time.Duration(rand.Int63n(int64(sleep)/10))
		time.Sleep(sleep + jitter)

		// Exponential Backoff
		sleep *= 2
	}
	return errors.New("max attempts reached")
}

func main() {
	rand.Seed(time.Now().UnixNano())
	retryWithBackoff(5, 100*time.Millisecond)
}
```

### Go Implementation: Circuit Breaker
Using `github.com/sony/gobreaker`.

```go
package main

import (
	"fmt"
	"time"

	"github.com/sony/gobreaker"
)

var cb *gobreaker.CircuitBreaker

func init() {
	settings := gobreaker.Settings{
		Name:        "HTTPClient",
		MaxRequests: 3,                 // Half-open -> Open after 3 requests
		Interval:    5 * time.Second,   // Clear counts every 5s
		Timeout:     10 * time.Second,  // Open -> Half-open after 10s
		ReadyToTrip: func(counts gobreaker.Counts) bool {
			failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
			return counts.Requests >= 3 && failureRatio >= 0.6
		},
	}
	cb = gobreaker.NewCircuitBreaker(settings)
}

func executeRequest() (interface{}, error) {
	return cb.Execute(func() (interface{}, error) {
		// Simulate request
		// resp, err := http.Get("http://example.com")
		return nil, fmt.Errorf("service unavailable") // Simulating failure
	})
}

func main() {
	for i := 0; i < 5; i++ {
		_, err := executeRequest()
		fmt.Printf("Call %d: %v\n", i, err)
		time.Sleep(1 * time.Second)
	}
}
```

## Interview Questions

**Q: What is the purpose of 'Jitter' in retry logic?**
**A:** Without jitter, if a service goes down, all clients might retry at the exact same intervals (e.g., exactly 1s, then 2s). This creates synchronized waves of traffic that hammer the server again. Jitter adds randomness to desynchronize the clients, smoothing out the load.

**Q: Explain the states of a Circuit Breaker.**
**A:**
1.  **Closed**: Normal operation. Requests go through.
2.  **Open**: The failure threshold was breached. Requests fail immediately (Fast Fail) without calling the backend.
3.  **Half-Open**: After a timeout, the breaker lets a few "test" requests through. If they succeed, it goes back to **Closed**. If they fail, it goes back to **Open**.

**Q: When is retrying NOT safe?**
**A:** Retrying is unsafe for non-idempotent operations. For example, retrying a `POST /payment` request might result in charging the user twice if the first request actually succeeded but the response timed out. Only retry if the operation is idempotent or if you know the request never reached the server.
