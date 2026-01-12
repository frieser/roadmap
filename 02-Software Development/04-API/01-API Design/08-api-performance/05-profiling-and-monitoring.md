# Profiling and Monitoring

## Summary
Profiling and Monitoring are distinct but complementary practices. **Monitoring** provides a high-level overview of system health over time (metrics, logs, dashboards), answering "Is the system healthy?". **Profiling** provides a low-level analysis of code execution (CPU, memory, goroutines) to identify bottlenecks, answering "Why is it slow?".

## Detailed Explanation

### Monitoring (Observability)
The three pillars of observability:
1.  **Metrics**: Aggregatable data (counters, gauges, histograms).
    *   *Tools*: Prometheus, Grafana, Datadog.
2.  **Logs**: Discrete events (info, error, debug).
    *   *Tools*: ELK Stack (Elasticsearch, Logstash, Kibana), Fluentd.
3.  **Traces**: The lifecycle of a request across distributed systems.
    *   *Tools*: Jaeger, Zipkin, OpenTelemetry.

### Profiling in Go (pprof)
Go has a built-in profiler called `pprof` that is incredibly powerful.
*   **CPU Profile**: Where is the CPU spending time?
*   **Heap Profile**: Where is memory allocated? (Detect leaks).
*   **Goroutine Profile**: Stack traces of all current goroutines. (Detect deadlocks/leaks).
*   **Block/Mutex Profile**: Where are goroutines waiting on synchronization primitives?

### Go Implementation: Enabling pprof
It's as simple as importing `net/http/pprof`.

```go
package main

import (
	"log"
	"net/http"
	_ "net/http/pprof" // Automatically registers handlers at /debug/pprof/
	"time"
)

func heavyWork() {
	for {
		// Simulate CPU load
		for i := 0; i < 1000000; i++ {
			_ = i * i
		}
		time.Sleep(100 * time.Millisecond)
	}
}

func main() {
	go heavyWork()

	log.Println("Server starting on :6060 (profiling port)")
	// It's best practice to run pprof on a separate port/server to avoid exposing it publicly
	log.Fatal(http.ListenAndServe("localhost:6060", nil))
}
```

### Analyzing Profiles
Once running, you can grab a snapshot:
```bash
# Interactive shell
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# Web UI visualization (requires Graphviz)
go tool pprof -http=:8081 http://localhost:6060/debug/pprof/heap
```

### Structured Logging
Avoid `fmt.Printf` or basic `log` in production. Use structured loggers like **Zap** or **Zerolog** for machine-readable logs (JSON) that can be easily queried.

```go
package main

import (
	"go.uber.org/zap"
)

func main() {
	logger, _ := zap.NewProduction()
	defer logger.Sync()

	url := "http://example.com"
	// Structured logging with context
	logger.Info("failed to fetch URL",
		zap.String("url", url),
		zap.Int("attempt", 3),
		zap.Duration("backoff", 100),
	)
}
```

## Interview Questions

**Q: When would you use a CPU profile vs. a Block profile?**
**A:** Use a **CPU profile** when your application is consuming high CPU or processing requests slowly due to computation. Use a **Block profile** when your application seems idle (low CPU) but is unresponsive or slow, which indicates goroutines are stuck waiting on channels, mutexes, or network I/O.

**Q: Why is structured logging important?**
**A:** Standard text logs are hard to parse programmatically. Structured logs (JSON) allow log aggregation systems (like Elasticsearch or Splunk) to index fields (e.g., `user_id`, `status_code`, `latency`). This enables powerful queries like "Show me all errors for user_id=123" or "Graph average latency over time".

**Q: What is OpenTelemetry?**
**A:** OpenTelemetry (OTel) is an open-source observability framework for generating, collecting, and exporting telemetry data (metrics, logs, and traces). It provides a vendor-neutral standard, allowing you to switch backends (e.g., from Jaeger to Datadog) without rewriting your instrumentation code.
