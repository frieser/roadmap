---
---

## Summary
The Circuit Breaker pattern prevents an application from repeatedly trying to execute an operation that's likely to fail. It acts like an electrical circuit breaker: when a service fails too many times (e.g., timeouts), the breaker "trips" (opens), instantly blocking all further requests to that service for a period. This allows the failing service time to recover and prevents cascading failures.

## Detailed Explanation

### States
1.  **Closed (Normal)**: Requests are allowed through. If requests succeed, it stays closed. If failures exceed a threshold (e.g., 50% errors), it trips to **Open**.
2.  **Open (Tripped)**: All requests fail immediately (Fail Fast) without calling the downstream service. The system waits for a timeout period (e.g., 60 seconds).
3.  **Half-Open (Recovery)**: After the timeout, a limited number of "test" requests are allowed through.
    *   If they succeed: The breaker resets to **Closed**.
    *   If they fail: The breaker goes back to **Open**.

### Why it's critical
Without a circuit breaker, if Service A depends on Service B, and Service B becomes unresponsive:
*   Service A threads hang waiting for Service B (timeouts).
*   Service A runs out of resources (threads/memory).
*   Service A crashes, taking down the whole system (Cascading Failure).

## Go-Specific Context/Examples

In Go, `sony/gobreaker` is a popular library implementation.

### Example: Using `sony/gobreaker`

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"time"

	"github.com/sony/gobreaker"
)

var cb *gobreaker.CircuitBreaker

func init() {
	settings := gobreaker.Settings{
		Name:        "HTTPClient",
		MaxRequests: 3,                // Max requests in Half-Open state
		Interval:    5 * time.Second,  // Window to count failures
		Timeout:     10 * time.Second, // Time to stay Open before Half-Open
		ReadyToTrip: func(counts gobreaker.Counts) bool {
			failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
			// Trip if > 3 requests and > 60% failed
			return counts.Requests >= 3 && failureRatio >= 0.6
		},
	}
	cb = gobreaker.NewCircuitBreaker(settings)
}

func GetURL(url string) ([]byte, error) {
	// Wrap the risky operation in cb.Execute
	body, err := cb.Execute(func() (interface{}, error) {
		resp, err := http.Get(url)
		if err != nil {
			return nil, err
		}
		defer resp.Body.Close()
		
		if resp.StatusCode >= 500 {
			return nil, fmt.Errorf("server error: %d", resp.StatusCode)
		}
		
		return ioutil.ReadAll(resp.Body)
	})

	if err != nil {
		return nil, err
	}
	return body.([]byte), nil
}

func main() {
	// Call a potentially flaky URL
	data, err := GetURL("http://localhost:8081/flaky")
	if err != nil {
		fmt.Println("Error:", err) // Returns "circuit breaker is open" immediately if tripped
	} else {
		fmt.Println("Success:", string(data))
	}
}
```

## Interview Questions

**Q: What is the "Fail Fast" concept?**
**A:** Fail Fast means reporting the error immediately instead of attempting an operation that is known to fail. In the context of Circuit Breakers, when the state is "Open", the application returns an error instantly rather than waiting 30 seconds for a connection timeout, saving resources.

**Q: How does the "Half-Open" state work?**
**A:** It acts as a probe or canary. After the "Open" timeout expires, the breaker tentatively lets a few requests through to test the waters. If the service has recovered, they succeed, and the breaker closes. If not, it re-opens. This prevents flooding a recovering service with full traffic immediately.

**Q: Should you use a Circuit Breaker for local database calls?**
**A:** Generally, yes. Even local databases can lock up or become overloaded. Wrapping DB calls in a circuit breaker ensures that if the DB is down, your web server doesn't hang all its threads waiting for connections, allowing it to potentially serve cached data or a friendly error page.
