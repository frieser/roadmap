---
---

## Summary
Cloud Design Patterns are essential blueprints for building resilient, scalable, and maintainable distributed systems. This note covers three critical categories: **Reliability Patterns** (Retry, Circuit Breaker, Bulkhead) to handle failures gracefully; **Structural Patterns** (Sidecar, Ambassador, Adapter) for modularizing peripheral concerns; and **Migration Patterns** (Strangler Fig) for incrementally modernizing legacy systems.

## Detailed Explanation

### 1. Reliability Patterns
Reliability patterns focus on making systems fault-tolerant and resilient to transient or permanent failures in distributed environments.

#### **Retry (Exponential Backoff & Jitter)**
*   **Concept**: Automatically retries a failed operation that is likely to be transient (e.g., network glitch).
*   **Exponential Backoff**: Increasing the wait time between retries exponentially (e.g., 1s, 2s, 4s, 8s) to avoid overwhelming the downstream service.
*   **Jitter**: Adding randomness to the backoff time to prevent "thundering herd" problems where multiple clients retry at the exact same moment.

#### **Circuit Breaker**
*   **Concept**: Prevents an application from repeatedly trying to execute an operation that is likely to fail, shielding the system from cascading failures.
*   **States**:
    *   **Closed**: Requests flow normally. Failures are tracked.
    *   **Open**: Threshold reached; requests fail immediately without calling the service.
    *   **Half-Open**: After a timeout, allow a limited number of test requests to see if the service has recovered.

#### **Bulkhead**
*   **Concept**: Isolates elements of an application into pools so that if one fails, the others continue to function.
*   **Analogy**: Ships are divided into watertight compartments (bulkheads) so a leak in one doesn't sink the entire vessel.
*   **Implementation**: Using separate thread pools or connection pools for different services.

---

### 2. Structural Patterns
Structural patterns define how containers or processes relate to each other in a microservices architecture.

#### **Sidecar**
*   **Concept**: Attaches peripheral tasks to a primary application in a separate process or container (the "sidecar").
*   **Usage**: Logging, monitoring, configuration, or network proxying (e.g., Envoy in Istio).
*   **Benefit**: Homogeneous management of cross-cutting concerns regardless of the primary app's language.

#### **Ambassador**
*   **Concept**: A specialized sidecar that acts as a proxy for network requests *from* the application.
*   **Usage**: Offloading retry logic, monitoring, or security (TLS termination) to the ambassador so the application only talks to `localhost`.

#### **Adapter**
*   **Concept**: A sidecar that standardizes the interface *to* the application.
*   **Usage**: Exposing a consistent monitoring interface (e.g., Prometheus metrics) for diverse legacy applications that have different internal formats.

---

### 3. Migration Patterns
Migration patterns help transition from legacy architectures (Monoliths) to modern ones (Microservices).

#### **Strangler Fig Pattern**
*   **Concept**: Incrementally replace specific pieces of functionality with new services until the old system is "strangled" and can be retired.
*   **Mechanism**: A routing facade (proxy) intercepts requests and directs them to either the legacy system or the new microservice based on the migrated features.
*   **Benefit**: Reduces risk by avoiding "big bang" rewrites.

---

## Go Implementation: Circuit Breaker

The following example uses the popular `sony/gobreaker` library to implement a circuit breaker for an external API call.

```go
package main

import (
	"errors"
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
		MaxRequests: 3,                // Max requests in Half-Open state
		Interval:    5 * time.Second,  // Cyclic period for Closed state to clear counts
		Timeout:     10 * time.Second, // Time in Open state before switching to Half-Open
		ReadyToTrip: func(counts gobreaker.Counts) bool {
			failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
			return counts.Requests >= 3 && failureRatio >= 0.6
		},
		OnStateChange: func(name string, from gobreaker.State, to gobreaker.State) {
			fmt.Printf("Circuit Breaker '%s' changed from %s to %s\n", name, from, to)
		},
	}
	cb = gobreaker.NewCircuitBreaker(settings)
}

func FetchData(url string) ([]byte, error) {
	// Execute wraps the call with Circuit Breaker logic
	body, err := cb.Execute(func() (interface{}, error) {
		resp, err := http.Get(url)
		if err != nil {
			return nil, err
		}
		defer resp.Body.Close()

		if resp.StatusCode != http.StatusOK {
			return nil, errors.New("external service error")
		}

		return io.ReadAll(resp.Body)
	})

	if err != nil {
		return nil, err
	}

	return body.([]byte), nil
}

func main() {
	// Example usage
	body, err := FetchData("https://api.example.com/data")
	if err != nil {
		if err == gobreaker.ErrOpenState {
			fmt.Println("Circuit is OPEN, request blocked.")
		} else {
			fmt.Printf("Error: %v\n", err)
		}
		return
	}
	fmt.Printf("Data fetched: %d bytes\n", len(body))
}
```

---

## Interview Questions

**Q: What is the difference between the Circuit Breaker and Retry patterns?**
**A:** Retry aims to recover from transient failures by attempting the operation again. Circuit Breaker aims to prevent the system from attempting an operation that is likely to fail (preventing resource exhaustion and cascading failures). Usually, Retry is used *inside* a Circuit Breaker or as part of the fallback logic.

**Q: How does the Sidecar pattern help in a Polyglot architecture?**
**A:** It allows infrastructure concerns (logging, service discovery, TLS) to be implemented once in a sidecar (often written in a high-performance language like Go or Rust) and shared across all services regardless of whether they are written in Java, Python, or Node.js.

**Q: Why is "Jitter" important in Exponential Backoff?**
**A:** Without jitter, multiple instances of a service failing at the same time will retry at the exact same intervals, causing synchronized spikes of traffic (thundering herd) that can further crash the downstream service. Jitter spreads these retries over time.

**Q: When should you NOT use the Strangler Fig pattern?**
**A:** If the legacy system is small enough for a quick rewrite, or if the internal dependencies are so tightly coupled that a clean "interceptor/proxy" cannot be placed between components without massive effort.

**Q: What is the primary purpose of the Bulkhead pattern?**
**A:** Fault isolation. It ensures that a failure in one service (or a specific component) does not consume all system resources (like threads or memory), allowing other unrelated parts of the system to remain responsive.
