---
---

# Datadog

Datadog is a cloud-scale monitoring and analytics platform (SaaS). It unifies infrastructure monitoring, application performance monitoring (APM), and log management into a single pane of glass.

## Summary

Datadog uses a **Push-based** agent architecture. The **Datadog Agent** runs on your hosts (or as a DaemonSet in K8s), collects metrics/logs/traces, and pushes them to the Datadog cloud. It is famous for its ease of use, zero-configuration integrations, and powerful tagging system.

## Detailed Explanation

### 1. Architecture
*   **Datadog Agent**: A complex binary (written in Go) that runs collectors. It acts as a local forwarder.
*   **DogStatsD**: An extension of the StatsD protocol. Apps send metrics to the local Agent via UDP. The Agent buffers and aggregates them before sending to the cloud API.
*   **Tagging**: The superpower of Datadog. Everything is tagged (`env:prod`, `region:us-east-1`, `service:login`). You slice and dice data dynamically by tags, not by hostnames.

### 2. Components
*   **Infrastructure**: CPU, Memory, Disk, Network.
*   **APM (Tracing)**: Distributed tracing for microservices.
*   **Logs**: Log ingestion and indexing.
*   **Synthetics**: Uptime checks from around the world.

---

## Go Implementation Example

Using `github.com/DataDog/datadog-go/v5/statsd` to send custom metrics via DogStatsD.

```go
package main

import (
	"log"
	"time"

	"github.com/DataDog/datadog-go/v5/statsd"
)

func main() {
	// 1. Create the client (connects to local agent on 8125)
	client, err := statsd.New("127.0.0.1:8125",
		statsd.WithNamespace("myapp."),    // Prefix for all metrics
		statsd.WithTags([]string{"env:prod", "role:worker"}), // Global tags
	)
	if err != nil {
		log.Fatal(err)
	}

	// 2. Send Metrics
	for {
		// Increment a counter
		client.Incr("requests.processed", []string{"status:ok"}, 1)

		// Set a gauge (e.g., queue size)
		client.Gauge("queue.size", 42, nil, 1)

		// Measure duration (Histogram/Timer)
		start := time.Now()
		time.Sleep(150 * time.Millisecond)
		client.Timing("job.duration", time.Since(start), nil, 1)

		// Send an Event (visible in Event Stream)
		client.SimpleEvent("Job Completed", "Batch #55 finished successfully")

		time.Sleep(1 * time.Second)
	}
}
```

## Interview Questions

**Q: What is the difference between StatsD and DogStatsD?**
**A:** StatsD is a standard protocol for sending metrics. DogStatsD is Datadog's extension of it. The key addition is **Tags**.
*   StatsD: `users.login.success:1|c` (Hierarchy based)
*   DogStatsD: `users.login:1|c|#status:success,region:us` (Tag based). Tags allow multidimensional queries (group by region, filter by status) without creating explosion of metric names.

**Q: How does Datadog APM work with Go?**
**A:** You use the `dd-trace-go` library. It wraps standard libraries (like `net/http`, `database/sql`, `gorilla/mux`). When a request comes in, the tracer generates a Span ID and Trace ID, propagates them through context, and sends the trace data asynchronously to the Datadog Agent, which forwards it to the cloud.

**Q: What is a "Monitor" in Datadog?**
**A:** A Monitor is an alerting rule. It queries the metric stream (e.g., `avg(last_5m):avg:system.cpu.idle{host:web*} < 10`). If the condition is met, it triggers a notification to Slack, PagerDuty, or email. It supports "Multi-Alerts" where one monitor config can alert separately for each tag group (e.g., alert if *any* single host goes high CPU).
