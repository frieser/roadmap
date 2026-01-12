---
---

## Summary
Short Polling (or simply "Polling") is the traditional technique where a client repeatedly sends HTTP requests to a server at fixed intervals (e.g., every 5 seconds) to check for new data. It is the simplest form of "real-time" data fetching but is inefficient due to unnecessary network traffic and latency.

## Detailed Explanation

### The Mechanism
1.  **Client**: Sets a timer (e.g., `setInterval` in JS).
2.  **Request**: Every X seconds, the client sends a GET request to the server.
3.  **Server**: Checks database/state.
    *   If data exists: Returns data (200 OK).
    *   If no data: Returns empty/same data (200 OK or 304 Not Modified).
4.  **Repeat**: The cycle continues indefinitely.

### Drawbacks
*   **Latency**: Data is only as "fresh" as the polling interval. If the interval is 10s, data could be 9.9s old.
*   **Waste**: If data updates rarely, 99% of requests are wasted, consuming bandwidth and server CPU just to say "nothing new".
*   **Server Load**: Scales linearly with the number of clients x frequency. 1000 users polling every 1s = 1000 RPS.

### Use Cases
Despite drawbacks, it is useful when:
*   Real-time requirements are loose (e.g., dashboard updating every 1 min).
*   Simplicity is key (no WebSocket infrastructure needed).
*   Caching is effective (HTTP caching works well here).

## Go-Specific Context/Examples

In Go, the server side is just a standard HTTP handler. The client side (or a Go service polling another service) uses a `time.Ticker`.

### Example: Go Client Polling a Service

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"time"
)

func main() {
	ticker := time.NewTicker(2 * time.Second)
	defer ticker.Stop()

	client := &http.Client{Timeout: 5 * time.Second}
	url := "http://localhost:8080/status"

	fmt.Println("Starting poller...")

	// Loop forever (or until a condition)
	for t := range ticker.C {
		fmt.Printf("Polling at %v: ", t.Format(time.TimeOnly))
		
		resp, err := client.Get(url)
		if err != nil {
			fmt.Printf("Error: %v\n", err)
			continue
		}
		
		body, _ := io.ReadAll(resp.Body)
		resp.Body.Close()
		
		fmt.Printf("Status: %s\n", string(body))
	}
}
```

### Server Side (Standard HTTP)
No special logic is needed, but using `ETag` or `Last-Modified` headers helps reduce bandwidth by allowing clients to receive `304 Not Modified`.

```go
func statusHandler(w http.ResponseWriter, r *http.Request) {
    // Simple response
    w.Write([]byte("All Systems Operational"))
}
```

## Interview Questions

**Q: When is Short Polling preferred over WebSockets?**
**A:** When data updates are infrequent (e.g., every 10 mins), "near" real-time isn't critical, and you want to leverage standard HTTP caching and stateless architecture. It is also much easier to scale via standard load balancers and CDNs than WebSockets.

**Q: How can you optimize Short Polling?**
**A:**
1.  **Jitter**: Add random randomization to the polling interval so all clients don't hit the server at the exact same millisecond (Thundering Herd).
2.  **Backoff**: Increase the interval if the server returns no new data repeatedly (Adaptive Polling).
3.  **HTTP Caching**: Use ETags so the server returns empty 304 responses instead of full payloads.

**Q: Calculate the load: 10,000 users polling every 5 seconds.**
**A:** 10,000 users / 5 seconds = 2,000 Requests Per Second (RPS). This is a significant constant load for a database or backend, mostly for empty checks.
