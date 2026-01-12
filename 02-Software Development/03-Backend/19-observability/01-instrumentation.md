#Observability
---
---

## Summary
Instrumentation is the process of adding code to your software to gather data about its behavior and performance. It acts as the sensors of your application, generating the raw data (logs, metrics, and traces) that monitoring tools analyze to provide observability. Without instrumentation, your application is a "black box."

## Detailed Explanation

### Types of Instrumentation
1.  **Black-box Monitoring**: Measuring from the outside (e.g., Pinging the server, checking HTTP 200 OK status). No code changes needed.
2.  **White-box Monitoring**: Measuring from the inside (e.g., Code reporting "I processed 500 orders" or "Database query took 10ms"). Requires Instrumentation.

### Methodologies
*   **The RED Method** (for Microservices/APIs):
    *   **R**ate: Number of requests per second.
    *   **E**rrors: Number of failed requests per second.
    *   **D**uration: Amount of time requests take (Latency).
*   **The USE Method** (for Infrastructure/Hardware):
    *   **U**tilization: % of time busy (CPU at 90%).
    *   **S**aturation: Amount of work queued/waiting.
    *   **E**rrors: Hardware error counts.

### Signals
Instrumentation code typically emits:
*   **Counters**: "Total requests", "Total errors" (always goes up).
*   **Gauges**: "Current memory usage", "Active goroutines" (goes up and down).
*   **Histograms**: "Request latency distribution" (99th percentile).

## Go-Specific Context/Examples

In Go, `prometheus/client_golang` is the standard library for instrumenting metrics.

### Example: Instrumenting an HTTP Handler

```go
package main

import (
	"net/http"
	"time"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

// Define a Counter metric (RED: Rate)
var requestsTotal = prometheus.NewCounterVec(
	prometheus.CounterOpts{
		Name: "http_requests_total",
		Help: "Total number of HTTP requests",
	},
	[]string{"path", "method"},
)

// Define a Histogram metric (RED: Duration)
var requestDuration = prometheus.NewHistogramVec(
	prometheus.HistogramOpts{
		Name:    "http_request_duration_seconds",
		Help:    "Duration of HTTP requests",
		Buckets: prometheus.DefBuckets,
	},
	[]string{"path"},
)

func init() {
	// Register metrics with Prometheus
	prometheus.MustRegister(requestsTotal)
	prometheus.MustRegister(requestDuration)
}

func instrumentedHandler(w http.ResponseWriter, r *http.Request) {
	start := time.Now()

	// Logic...
	w.Write([]byte("Hello, Observability!"))

	// Record Metrics
	duration := time.Since(start).Seconds()
	requestsTotal.WithLabelValues(r.URL.Path, r.Method).Inc()
	requestDuration.WithLabelValues(r.URL.Path).Observe(duration)
}

func main() {
	http.HandleFunc("/", instrumentedHandler)
	
	// Expose metrics endpoint for Prometheus to scrape
	http.Handle("/metrics", promhttp.Handler())
	
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: What is the difference between Instrumentation and Monitoring?**
**A:** Instrumentation is the *act* of adding code to generate data. Monitoring is the *process* of collecting, aggregating, and displaying that data to understand system health. You cannot have effective white-box monitoring without instrumentation.

**Q: Why should you avoid high-cardinality labels in instrumentation?**
**A:** High cardinality means a label has too many unique values (e.g., putting `UserID` or `IPAddress` as a metric label). This creates a unique time series for *every user*, exploding memory usage in the metrics database (Prometheus) and slowing down queries. Metrics should aggregate data (e.g., by `status_code` or `endpoint`); logs are for specific details like UserIDs.

**Q: What is "Automatic Instrumentation" (Auto-instrumentation)?**
**A:** It allows capturing telemetry (like traces) without modifying the source code manually, often using agents or eBPF (in Linux). While popular in Java/Python, Go typically requires manual instrumentation (or library wrappers) because it compiles to a binary, though OpenTelemetry is improving Go auto-instrumentation capabilities.
