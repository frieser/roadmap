#Observability
---
---

## Summary
Monitoring is the systematic process of collecting, analyzing, and using information to track the progress and health of an application or infrastructure. While **Observability** asks "Why is my system behaving this way?", **Monitoring** asks "Is my system healthy?" by checking against predefined thresholds.

## Detailed Explanation

### Components of Monitoring
1.  **Metrics Collection**: Gathering numeric data points over time (CPU usage, Request Count).
2.  **Visualization (Dashboards)**: Graphical representation of metrics (Grafana boards) to spot trends.
3.  **Alerting**: Sending notifications (PagerDuty, Slack) when a metric violates a threshold (e.g., "Error rate > 5% for 2 minutes").
4.  **Health Checks**: Binary checks to see if a service is "alive" (Liveness) and "ready to accept traffic" (Readiness).

### What to Monitor?
*   **Infrastructure**: CPU, RAM, Disk I/O, Network.
*   **Application**: HTTP Error rates, Latency, Throughput.
*   **Business**: Orders per minute, New user signups.

### Alerting Philosophy
*   **Alert on Symptoms, not Causes**: Alert on "High Latency" or "High Error Rate" (what the user sees), not on "High CPU" (which might be fine).
*   **Actionable**: Every alert should require a human action. If not, it's noise.

## Go-Specific Context/Examples

A standard pattern in Go services is exposing a `/healthz` or `/status` endpoint for load balancers (like Kubernetes) and a `/metrics` endpoint for Prometheus.

### Example: Health Check & Metrics Export

```go
package main

import (
	"database/sql"
	"fmt"
	"net/http"

	_ "github.com/lib/pq"
)

var db *sql.DB

// Simple Health Check (Liveness)
func healthz(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)
	w.Write([]byte("ok"))
}

// Deep Health Check (Readiness) - Checks dependencies
func readyz(w http.ResponseWriter, r *http.Request) {
	if err := db.Ping(); err != nil {
		http.Error(w, "Database unavailable", http.StatusServiceUnavailable)
		return
	}
	w.WriteHeader(http.StatusOK)
	w.Write([]byte("ready"))
}

func main() {
	// Setup DB...
	
	http.HandleFunc("/healthz", healthz) // Is the process running?
	http.HandleFunc("/readyz", readyz)   // Can we serve traffic?
	
	// In a real app, you'd bind this to a different port or internal interface
	fmt.Println("Monitoring endpoints running on :8080")
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: What is the difference between Liveness and Readiness probes?**
**A:**
*   **Liveness**: "Is the process running?" If this fails, the orchestrator (K8s) kills and restarts the container. (Example: Deadlock detection).
*   **Readiness**: "Can it serve traffic?" If this fails, the load balancer stops sending traffic to this instance, but doesn't kill it. (Example: Waiting for DB connection, or warming up cache).

**Q: Explain "Four Golden Signals" of monitoring.**
**A:** A framework by Google SRE:
1.  **Latency**: Time taken to service a request.
2.  **Traffic**: Demand on the system (req/sec).
3.  **Errors**: Rate of requests that fail.
4.  **Saturation**: How "full" the service is (queue depth, memory usage).

**Q: Push vs Pull Monitoring?**
**A:**
*   **Pull (Prometheus)**: The monitoring system scrapes metrics from the app. Better for knowing *if* an app is down (scrape fails).
*   **Push (Graphite/Datadog)**: The app sends metrics to the monitoring system. Better for short-lived jobs or batch processes that might finish before a scrape occurs.
