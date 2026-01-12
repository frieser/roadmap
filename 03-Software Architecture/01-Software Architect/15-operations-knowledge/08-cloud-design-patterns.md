---
---

## Summary
Cloud Design Patterns are reusable solutions to common problems encountered when building distributed, scalable, and reliable applications in the cloud. They address challenges like latency, failure management, state synchronization, and resource optimization.

## Detailed Explanation

### Common Patterns
1.  **Ambassador / Sidecar**:
    *   Helper container running alongside the main app. Handles logging, monitoring, or networking (Service Mesh proxy).
2.  **Circuit Breaker**:
    *   Prevents cascading failures by stopping calls to a failing service.
3.  **CQRS (Command Query Responsibility Segregation)**:
    *   Separates read (Query) and write (Command) models. Allows scaling reads independently of writes (e.g., Read from Cache/Replica, Write to Master).
4.  **Event Sourcing**:
    *   Stores state as a sequence of events (e.g., "ItemAdded", "ItemRemoved") rather than just the current state.
5.  **Strangler Fig**:
    *   Incrementally migrates a legacy system by replacing specific functionalities with new microservices, eventually "strangling" the old system.
6.  **Retry with Backoff**:
    *   Automatically retrying failed operations (transient faults) with increasing delays (Exponential Backoff).

## Go-Specific Context/Examples

### Example: Sidecar Pattern (Conceptual)
In Go/Kubernetes, you don't write the sidecar logic *in* the app, but you design the app to rely on it.
*   **App**: Sends HTTP request to `localhost:3500` (The Sidecar).
*   **Sidecar (Dapr/Envoy)**: Handles mTLS, tracing, and routing to the actual destination.

### Example: Retry with Exponential Backoff in Go

```go
package main

import (
	"fmt"
	"math"
	"time"
)

func retryOperation(attempts int, sleep time.Duration, f func() error) error {
	for i := 0; i < attempts; i++ {
		err := f()
		if err == nil {
			return nil
		}
		
		fmt.Printf("Attempt %d failed: %v. Retrying...\n", i+1, err)
		
		// Exponential Backoff: 1s, 2s, 4s, 8s...
		backoff := float64(sleep) * math.Pow(2, float64(i))
		time.Sleep(time.Duration(backoff))
	}
	return fmt.Errorf("after %d attempts, last error: %s", attempts, "failed")
}

func main() {
	// Simulate a flaky function
	flakyOp := func() error {
		return fmt.Errorf("connection timeout")
	}

	err := retryOperation(3, 1*time.Second, flakyOp)
	if err != nil {
		fmt.Println("Operation permanently failed:", err)
	}
}
```

## Interview Questions

**Q: When should you use the Strangler Fig pattern?**
**A:** When migrating a large Monolith to Microservices. Instead of a "Big Bang" rewrite (high risk), you put a proxy in front. You build one new microservice for a specific feature (e.g., "Search"). The proxy routes "Search" traffic to the new service and everything else to the old Monolith. You repeat this until the Monolith is empty.

**Q: What is the main disadvantage of CQRS?**
**A:** **Complexity** and **Eventual Consistency**. You now have two models to maintain. The "Read" database might be slightly out of sync with the "Write" database, which adds complexity to the UI/UX (user saves data but doesn't see it immediately).

**Q: Why use a Sidecar instead of a library?**
**A:** A Sidecar is **language agnostic**. You can write your app in Go, Python, or Java, and they all use the same Sidecar (e.g., Envoy) for logging and mTLS. It decouples infrastructure concerns from application logic.
