---
---

## Summary
The **Circuit Breaker** pattern is a stability pattern used in distributed systems to prevent an application from repeatedly trying to execute an operation that is likely to fail. It functions like an electrical circuit breaker: when the number of failures crosses a threshold, the circuit "trips" (opens), and further calls are immediately failed without attempting the operation, allowing the failing service time to recover.

## Detailed Explanation
In a microservices architecture, services often call other services. If a downstream service is slow or unavailable, upstream services can exhaust resources (threads, connections) waiting for it, leading to cascading failures.

The Circuit Breaker pattern prevents this by wrapping the dangerous operation in a proxy that monitors for failures. It operates as a State Machine with three states:

1.  **Closed**: The circuit is closed, and requests are allowed to pass through. If the failure rate or count exceeds a configured threshold, the circuit trips to the **Open** state.
2.  **Open**: The circuit is open, and requests fail immediately (fail-fast) without calling the downstream service. This prevents resource exhaustion and gives the downstream service time to recover. After a timeout period, the circuit moves to **Half-Open**.
3.  **Half-Open**: A limited number of "trial" requests are allowed through. 
    *   If they succeed, the circuit assumes the service has recovered and resets to **Closed**.
    *   If they fail, the circuit returns to **Open** and restarts the timeout.

### Key Benefits
*   **Fail Fast**: Prevents the calling service from waiting for timeouts.
*   **Resource Conservation**: Frees up threads and connections that would otherwise be blocked.
*   **Self-Healing**: Automatically detects when the downstream service recovers.

### Go Implementation
In Go, the `sony/gobreaker` library is a popular implementation.

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"time"

	"github.com/sony/gobreaker"
)

var cb *gobreaker.CircuitBreaker

func init() {
	settings := gobreaker.Settings{
		Name:        "HTTP GET",
		MaxRequests: 3,               // Max requests allowed in Half-Open state
		Interval:    5 * time.Second, // Cyclic period of the closed state to clear counts
		Timeout:     2 * time.Second, // Duration of Open state before moving to Half-Open
		ReadyToTrip: func(counts gobreaker.Counts) bool {
			// Trip the circuit if failure rate > 50% and we have at least 5 requests
			failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
			return counts.Requests >= 5 && failureRatio >= 0.5
		},
	}
	cb = gobreaker.NewCircuitBreaker(settings)
}

func GetServiceData() ([]byte, error) {
	// Wrap the request logic in the Execute method
	body, err := cb.Execute(func() (interface{}, error) {
		resp, err := http.Get("http://unstable-service/data")
		if err != nil {
			return nil, err
		}
		defer resp.Body.Close()

		if resp.StatusCode >= 500 {
			return nil, fmt.Errorf("server error: %d", resp.StatusCode)
		}

		return io.ReadAll(resp.Body)
	})

	if err != nil {
		// Circuit is open or request failed
		return nil, err
	}
	return body.([]byte), nil
}
```

## Interview Questions

**Q: What is the difference between the Retry pattern and the Circuit Breaker pattern?**
**A:** The **Retry** pattern assumes a failure is transient and immediately tries again (often with backoff), which is useful for momentary glitches. The **Circuit Breaker** pattern assumes a failure might be persistent and prevents further attempts for a period, protecting the system from overload. They are often combined: a few retries first, and if they all fail, the circuit breaker counts the failure.

**Q: What happens in the Half-Open state?**
**A:** The **Half-Open** state is a testing phase. It allows a limited number of requests to pass through to check if the underlying issue is resolved. If these requests succeed, the circuit closes (resumes normal operation); if they fail, it re-opens (pauses again). This prevents a recovered service from being immediately overwhelmed by a flood of pending requests ("thundering herd").

**Q: How should an application handle a request when the circuit is Open?**
**A:** The application should implement **Graceful Degradation**. This could involve returning a default/fallback value, serving stale data from a cache, or returning a user-friendly error message indicating the feature is temporarily unavailable, rather than just crashing or hanging.
