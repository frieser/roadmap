# API Performance Metrics

## Summary
API performance metrics are quantitative measures used to assess the efficiency, reliability, and speed of an API. Key metrics include **Latency** (response time), **Throughput** (requests per second), **Error Rate** (percentage of failed requests), and **Saturation** (resource utilization). Monitoring these metrics is crucial for identifying bottlenecks, ensuring SLAs are met, and maintaining a high-quality user experience.

## Detailed Explanation

### Core Metrics ( The "Golden Signals")
Google's SRE book defines four "Golden Signals" for monitoring distributed systems:

1.  **Latency**: The time it takes to service a request. It's crucial to distinguish between successful requests and failed ones.
    *   **Average vs. Percentiles**: Averages hide outliers. Use percentiles (p50, p95, p99) to understand the long tail of performance.
    *   *Example*: p99 latency of 500ms means 99% of requests are faster than 500ms.

2.  **Traffic (Throughput)**: A measure of how much demand is being placed on your system.
    *   Typically measured in Requests Per Second (RPS) or Queries Per Second (QPS).

3.  **Errors**: The rate of requests that fail.
    *   Explicit failures (HTTP 500s).
    *   Implicit failures (HTTP 200 OK but with wrong content).
    *   Policy failures (HTTP 429 Too Many Requests).

4.  **Saturation**: How "full" your service is.
    *   A measure of your system fraction, emphasizing the resources that are most constrained (e.g., CPU, Memory, I/O).

### Other Important Metrics
*   **Availability (Uptime)**: The percentage of time the API is operational and accessible.
*   **APDEX (Application Performance Index)**: An open standard for measuring performance of software applications.

### Measuring in Go
In Go, middleware is the standard way to intercept requests and measure these metrics. The `prometheus` client library is widely used to expose these metrics.

```go
package main

import (
	"log"
	"net/http"
	"strconv"
	"time"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

// Define metrics
var (
	httpRequestsTotal = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "http_requests_total",
			Help: "Total number of HTTP requests",
		},
		[]string{"path", "method", "status"},
	)
	httpRequestDuration = prometheus.NewHistogramVec(
		prometheus.HistogramOpts{
			Name:    "http_request_duration_seconds",
			Help:    "Duration of HTTP requests",
			Buckets: prometheus.DefBuckets,
		},
		[]string{"path", "method"},
	)
)

func init() {
	// Register metrics
	prometheus.MustRegister(httpRequestsTotal)
	prometheus.MustRegister(httpRequestDuration)
}

// responseWriter wraps http.ResponseWriter to capture status code
type responseWriter struct {
	http.ResponseWriter
	statusCode int
}

func (rw *responseWriter) WriteHeader(code int) {
	rw.statusCode = code
	rw.ResponseWriter.WriteHeader(code)
}

// Middleware to record metrics
func metricsMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		rw := &responseWriter{ResponseWriter: w, statusCode: http.StatusOK}
		
		next.ServeHTTP(rw, r)

		duration := time.Since(start).Seconds()
		status := strconv.Itoa(rw.statusCode)

		httpRequestsTotal.WithLabelValues(r.URL.Path, r.Method, status).Inc()
		httpRequestDuration.WithLabelValues(r.URL.Path, r.Method).Observe(duration)
	})
}

func main() {
	handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello, World!"))
	})

	http.Handle("/metrics", promhttp.Handler())
	http.Handle("/", metricsMiddleware(handler))

	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## Interview Questions

**Q: Why is p99 latency more important than average latency?**
**A:** Average latency can be skewed by a few very fast requests and hides the experience of the slowest users. p99 (99th percentile) tells you the maximum time 99% of your users are waiting, which better reflects the "worst-case" experience for the majority and helps identify tail latency issues.

**Q: How do you differentiate between 'Saturation' and 'Traffic'?**
**A:** Traffic (Throughput) is the external demand (e.g., 1000 RPS). Saturation is the internal capacity utilization (e.g., CPU at 90%). You can have high traffic with low saturation (efficient system) or low traffic with high saturation (inefficient or resource-constrained system).

**Q: What is the difference between white-box and black-box monitoring?**
**A:** White-box monitoring involves reporting metrics from the internals of the application (e.g., logs, profiling, Prometheus metrics). Black-box monitoring involves testing the externally visible behavior as a user would see it (e.g., pinging the endpoint to check availability).
