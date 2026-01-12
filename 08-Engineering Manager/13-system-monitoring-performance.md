# System Monitoring and Performance

## Summary
System Monitoring and Performance (often grouped under **Observability**) is the practice of understanding the internal state of a system based on its external outputs. For Engineering Managers, this is critical for reliability. You cannot manage what you cannot measure. The goal is to move from "users telling us it's down" to "fixing it before users notice."

## Detailed Explanation

### 1. Monitoring vs. Observability
*   **Monitoring**: "Is the system healthy?" (Dashboards, Green/Red lights). Tells you *that* something is wrong.
*   **Observability**: "Why is the system behaving this way?" (Querying data). Tells you *why* it is wrong.

### 2. The Three Pillars of Observability
1.  **Metrics**: Aggregated numbers over time (e.g., "CPU is at 80%", "Requests per second = 500"). Cheap to store, good for trends.
2.  **Logs**: Discrete events (e.g., "User X failed to login at 10:00 PM"). High detail, expensive to store.
3.  **Traces**: The lifecycle of a request as it hops through microservices. Essential for debugging latency in distributed systems.

### 3. SRE Terminology (SLI, SLO, SLA)
*   **SLI (Service Level Indicator)**: The metric. (e.g., "Latency").
*   **SLO (Service Level Objective)**: The goal. (e.g., "99% of requests < 200ms"). *This is an internal engineering target.*
*   **SLA (Service Level Agreement)**: The promise to the customer. (e.g., "If availability < 99.9%, we pay you back"). *This is a legal contract.*

### 4. Golden Signals (The RED Method)
For every service, you should measure:
*   **R**ate: Number of requests per second.
*   **E**rrors: Number of failed requests per second.
*   **D**uration: How long requests take (Distribution: P50, P90, P99).

### 5. Alerting Philosophy
*   **Alert on Symptoms, not Causes**: Alert on "High Error Rate" (Symptom), not "High CPU" (Cause). High CPU is fine if the users are happy.
*   **Alert Fatigue**: If the pager rings for non-urgent issues, engineers will ignore it. Delete flaky alerts.

## Go Code Example: Observability Middleware
This example demonstrates how to instrument a Go HTTP server to capture the "RED" metrics (Rate, Error, Duration) using a middleware pattern.

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

// MetricsSimulator represents a monitoring system (e.g., Prometheus)
type MetricsSimulator struct {
	RequestCount int
	ErrorCount   int
	Latencies    []time.Duration
}

var stats = MetricsSimulator{}

// ObservabilityMiddleware wraps a handler to capture metrics
func ObservabilityMiddleware(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		
		// Custom ResponseWriter to capture status code
		ww := &statusWriter{ResponseWriter: w, status: http.StatusOK}
		
		// Process Request
		next(ww, r)

		// Record Duration
		duration := time.Since(start)
		
		// Update Metrics (Thread-safety omitted for brevity)
		stats.RequestCount++
		stats.Latencies = append(stats.Latencies, duration)
		if ww.status >= 500 {
			stats.ErrorCount++
		}

		// Log structured event
		log.Printf("[HTTP] method=%s path=%s status=%d duration=%s", 
			r.Method, r.URL.Path, ww.status, duration)
	}
}

// statusWriter captures the HTTP status code
type statusWriter struct {
	http.ResponseWriter
	status int
}

func (w *statusWriter) WriteHeader(status int) {
	w.status = status
	w.ResponseWriter.WriteHeader(status)
}

func HelloHandler(w http.ResponseWriter, r *http.Request) {
	// Simulate work
	time.Sleep(50 * time.Millisecond)
	
	if r.URL.Query().Get("error") == "true" {
		http.Error(w, "Something went wrong", http.StatusInternalServerError)
		return
	}
	fmt.Fprintf(w, "Hello, World!")
}

func main() {
	http.HandleFunc("/", ObservabilityMiddleware(HelloHandler))

	// Background reporter
	go func() {
		for {
			time.Sleep(5 * time.Second)
			avgLatency := time.Duration(0)
			if len(stats.Latencies) > 0 {
				total := time.Duration(0)
				for _, l := range stats.Latencies {
					total += l
				}
				avgLatency = total / time.Duration(len(stats.Latencies))
			}
			
			fmt.Printf("\n--- Metrics Snapshot (Last 5s) ---\n")
			fmt.Printf("Rate: %d reqs\n", stats.RequestCount)
			fmt.Printf("Errors: %d\n", stats.ErrorCount)
			fmt.Printf("Avg Duration: %s\n", avgLatency)
			fmt.Println("----------------------------------")
		}
	}()

	fmt.Println("Server running on :8080...")
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

### Q: "Explain P99 Latency vs. Average Latency. Why does it matter?"
**A:**
*   **Average** hides outliers. If 99 requests take 1ms and 1 takes 10s, the average is ~100ms (looks okay).
*   **P99** (99th Percentile) tells you the experience of the slowest 1% of users. In the example above, P99 is 10s.
*   **Why**: The "tail latency" (P99) usually affects your most important/heavy users. Also, in microservices, tail latencies compound (a request hitting 5 services has a high chance of hitting a P99 outlier).

### Q: "What is an Error Budget?"
**A:**
*   It's the "allowable unreliability."
*   If SLO is 99.9%, the Error Budget is 0.1% (43 minutes/month).
*   **Usage**: If you have budget left, you can ship risky features fast. If you burned the budget (too many outages), you must freeze features and work only on stability until the budget resets.

### Q: "How do you solve Alert Fatigue?"
**A:**
*   **Audit**: Review every alert that fired last week. Was it actionable? If not, delete it or tune it.
*   **Severity Levels**: P1 (Wake up now), P2 (Next business day), P3 (Log only).
*   **Auto-healing**: If a script can fix it (e.g., restart pod), don't page a human.
