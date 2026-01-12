---
---

## Summary
Graceful Degradation is a fault-tolerance strategy where a system continues to function, albeit with reduced functionality or quality, in the event of a component failure, high load, or errors. Rather than crashing completely (catastrophic failure), the system "degrades" gracefully to provide the best possible user experience under the circumstances.

## Detailed Explanation

### Concept
If a non-essential dependency fails (e.g., the recommendation engine), the main application (e.g., the e-commerce checkout) should not break. Instead, it should hide the recommendation section or show generic items.

### Graceful Degradation vs Progressive Enhancement
*   **Graceful Degradation**: Build for the best environment/system, then ensure it breaks safely for worse ones.
*   **Progressive Enhancement**: Build for the baseline (lowest common denominator), then add features for better environments.

### Examples
1.  **UI**: If a custom font fails to load, fall back to a system font (`sans-serif`).
2.  **Backend**: If the Redis cache is down, fall back to querying the primary database (slower, but works), or return "stale" data.
3.  **Microservices**: If the "User Profile" service is slow, return the "Home Page" without the user's avatar, rather than a 500 error.

## Go-Specific Context/Examples

In Go, this is often implemented using the `Circuit Breaker` pattern, timeouts, or explicit error handling fallback logic.

### Example: Fallback Logic with Context Timeout

```go
package main

import (
	"context"
	"fmt"
	"time"
)

// Primary (Slow/Flaky) Service
func getRealTimeData(ctx context.Context) (string, error) {
	// Simulate work
	select {
	case <-time.After(2 * time.Second): // Takes 2s
		return "Live Stock Price: $150.00", nil
	case <-ctx.Done():
		return "", ctx.Err()
	}
}

// Fallback (Fast/Cached) Service
func getCachedData() string {
	return "Cached Price: $149.50 (Data from 5 mins ago)"
}

func handleRequest() {
	// We want an answer in 1 second max
	ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancel()

	fmt.Println("Fetching data...")

	// Try Primary
	data, err := getRealTimeData(ctx)
	if err != nil {
		// Graceful Degradation: Primary failed/timed out, use fallback
		fmt.Printf("Primary failed (%v). Degrading to fallback.\n", err)
		data = getCachedData()
	}

	fmt.Println("Result:", data)
}

func main() {
	handleRequest()
}
```

## Interview Questions

**Q: How does Graceful Degradation differ from Fault Tolerance?**
**A:** Fault tolerance usually implies the system continues to operate *normally* (zero impact) despite a fault (e.g., via redundancy/replica sets). Graceful degradation acknowledges the impact but mitigates it by offering a simplified or reduced service level rather than a complete outage.

**Q: Design a graceful degradation strategy for a video streaming site.**
**A:**
1.  If high-res (4K) bandwidth is unavailable, automatically downgrade to 1080p or 720p (Adaptive Bitrate).
2.  If the "Comments" microservice is down, load the video player but hide the comments section.
3.  If the CDN is unreachable in one region, route traffic to the next closest region (even if higher latency).

**Q: What is the "Circuit Breaker" pattern's role here?**
**A:** A Circuit Breaker prevents the application from repeatedly trying to execute an operation that's likely to fail (like calling a down service). By "opening" the circuit, it instantly fails fast or returns a fallback value (graceful degradation), allowing the struggling system time to recover.
