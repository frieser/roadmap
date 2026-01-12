---
---

# Availability Patterns

Availability patterns ensure that your application remains responsive and functional even when components fail or traffic spikes.

## Summary

*   **Circuit Breaker**: Prevents an application from repeatedly trying to execute an operation that's likely to fail.
*   **Health Endpoint Monitoring**: Exposing functional checks to allow external systems (Load Balancers, K8s) to verify app health.
*   **Throttling**: Limiting the consumption of resources by specific users or tenants.

---

## Go Implementation: Circuit Breaker

Using the popular `github.com/sony/gobreaker` library.

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
	var st gobreaker.Settings
	st.Name = "HTTP-GET"
	st.Timeout = time.Second * 2 // Time to wait in Open state before Half-Open
	st.ReadyToTrip = func(counts gobreaker.Counts) bool {
		// Trip if > 3 failures
		return counts.ConsecutiveFailures > 3
	}
	cb = gobreaker.NewCircuitBreaker(st)
}

func GetURL(url string) ([]byte, error) {
	// Wrap the risky operation in Execute
	body, err := cb.Execute(func() (interface{}, error) {
		resp, err := http.Get(url)
		if err != nil {
			return nil, err
		}
		defer resp.Body.Close()
		
		if resp.StatusCode >= 500 {
			return nil, fmt.Errorf("server error")
		}
		
		return io.ReadAll(resp.Body)
	})

	if err != nil {
		return nil, err
	}
	return body.([]byte), nil
}

func main() {
	// If the server is down, the CB will eventually "Trip" and stop making requests
	// immediately returning an error to save resources.
	for i := 0; i < 10; i++ {
		_, err := GetURL("http://localhost:9999") // Assume down
		fmt.Printf("Attempt %d: %v\n", i, err)
		time.Sleep(500 * time.Millisecond)
	}
}
```

## Interview Questions

**Q: Why use a Circuit Breaker?**
**A:** To fail fast. If a downstream service (e.g., Database) is down, your application shouldn't hang for 30 seconds waiting for a timeout on every request. This causes thread pool exhaustion and can crash the caller. A Circuit Breaker detects the failure and immediately returns an error, preserving system resources.

**Q: Explain the states of a Circuit Breaker.**
**A:**
1.  **Closed**: Normal operation. Requests pass through.
2.  **Open**: The failure threshold was reached. Requests fail immediately.
3.  **Half-Open**: After a timeout, allow a *limited* number of test requests. If they succeed, switch to **Closed**. If they fail, switch back to **Open**.
