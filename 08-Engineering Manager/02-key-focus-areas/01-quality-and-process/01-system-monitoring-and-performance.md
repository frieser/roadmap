---
---

## Summary
**System Monitoring and Performance** is the sensory nervous system of software engineering. For an Engineering Manager, it is about shifting from "reactive" fire-fighting to "proactive" observability. It involves defining clear Service Level Objectives (SLOs), measuring the "Golden Signals" (Latency, Traffic, Errors, Saturation), and ensuring the team builds systems that are debuggable by design.

## Detailed Explanation

### 1. Observability vs. Monitoring
*   **Monitoring**: Tells you *when* something is wrong (e.g., "CPU usage is 99%").
*   **Observability**: Tells you *why* something is wrong (e.g., "CPU is high because Function X is stuck in a loop"). It relies on three pillars: **Logs**, **Metrics**, and **Traces**.

### 2. The Golden Signals (Google SRE Book)
To effectively monitor a distributed system, track these four metrics:
1.  **Latency**: The time it takes to service a request. (Distinguish between successful and failed requests).
2.  **Traffic**: A measure of how much demand is being placed on your system (e.g., requests per second).
3.  **Errors**: The rate of requests that fail (e.g., HTTP 500s).
4.  **Saturation**: How "full" your service is (e.g., memory usage, thread pool capacity).

### 3. SLOs, SLIs, and SLAs
*   **SLI (Service Level Indicator)**: The metric (e.g., "Latency is 200ms").
*   **SLO (Service Level Objective)**: The internal goal (e.g., "99% of requests in < 300ms"). This is the team's target.
*   **SLA (Service Level Agreement)**: The external contract with users (e.g., "If downtime > 1%, we refund money").

### 4. Performance Culture
*   **Performance as a Feature**: Treat performance regressions as bugs.
*   **Budgets**: Establish error budgets. If the team burns through the budget (too many outages), feature work stops to focus on stability.

## Go Code Example: Middleware for Golden Signals
In Go, monitoring is often implemented via middleware that wraps HTTP handlers. This example demonstrates how to capture **Latency** and **Status Codes** (Errors) automatically for every request.

```go
package main

import (
	"log"
	"net/http"
	"time"
)

// statusRecorder wraps ResponseWriter to capture the HTTP status code
type statusRecorder struct {
	net/http.ResponseWriter
	StatusCode int
}

// WriteHeader captures the status code before writing it
func (rec *statusRecorder) WriteHeader(code int) {
	rec.StatusCode = code
	rec.ResponseWriter.WriteHeader(code)
}

// MonitorMiddleware captures Latency and Errors (Golden Signals)
func MonitorMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()

		// Wrap the writer to capture status code
		rec := &statusRecorder{ResponseWriter: w, StatusCode: http.StatusOK}
		
		// Process request
		next.ServeHTTP(rec, r)

		// Calculate Metrics
		duration := time.Since(start)
		
		// Log structured metric (simulating export to Prometheus/Datadog)
		log.Printf(
			"[METRIC] Path=%s | Method=%s | Status=%d | Latency=%v | UserAgent=%s",
			r.URL.Path,
			r.Method,
			rec.StatusCode,
			duration,
			r.UserAgent(),
		)

		// Alerting logic (simplified)
		if duration > 500*time.Millisecond {
			log.Println("[ALERT] High Latency detected!")
		}
		if rec.StatusCode >= 500 {
			log.Println("[ALERT] Server Error detected!")
		}
	})
}

func mainHandler(w http.ResponseWriter, r *http.Request) {
	time.Sleep(100 * time.Millisecond) // Simulate work
	w.Write([]byte("Hello, Observability!"))
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", mainHandler)

	// Wrap mux with monitoring
	wrappedMux := MonitorMiddleware(mux)

	log.Println("Server starting on :8080")
	http.ListenAndServe(":8080", wrappedMux)
}
```

## Interview Questions

### Q: "How do you decide what to monitor in a new system?"
**A:** Start with the user experience, not the infrastructure.
*   **Top-Down**: Monitor the "Golden Signals" first (Latency, Errors, Traffic). If the user sees 500 errors, that matters more than CPU usage.
*   **Business Metrics**: Monitor critical flows (e.g., "Checkout Success Rate").
*   **Refinement**: Add debug-level metrics (CPU, Memory, Garbage Collection) later for troubleshooting.

### Q: "Your system's latency has spiked. Walk me through your debugging process."
**A:**
1.  **Verify**: Check the dashboard to confirm the spike is real and affects users.
2.  **Isolate**: Is it a specific endpoint? A specific region? A recent deployment?
3.  **Trace**: Look at distributed traces (e.g., Jaeger) to see *where* time is being spent (Database? External API?).
4.  **Correlate**: Check logs for errors occurring at the same timestamp.

### Q: "Explain the concept of an Error Budget."
**A:** An Error Budget is 100% minus your SLO (e.g., if SLO is 99.9%, budget is 0.1%).
*   **Usage**: It aligns product and engineering. If we have budget left, we take risks and ship fast. If the budget is exhausted, we freeze features and fix reliability. It turns "reliability" from a vague argument into a quantifiable resource.
