---
---

# 01. Prometheus Deep Dive

## Summary
Prometheus is an open-source systems monitoring and alerting toolkit originally built at SoundCloud. It joined the CNCF in 2016 as the second hosted project after Kubernetes. It is the de-facto standard for metric-based monitoring in cloud-native environments.

## Detailed Development

### 1. Data Model: Time Series
Prometheus stores data as **Time Series**: streams of timestamped values belonging to the same metric and the same set of labeled dimensions.
*   **Format**: `<metric name>{<label name>=<label value>, ...} <value>`
*   **Example**: `http_requests_total{method="POST", endpoint="/api/login"} 1024`

### 2. Metric Types
1.  **Counter**: A cumulative metric that only goes up (e.g., total requests). Resets to zero on restart.
2.  **Gauge**: A single numerical value that can go up and down (e.g., memory usage, temperature).
3.  **Histogram**: Samples observations (usually request durations or response sizes) and counts them in configurable buckets. Used for p99.
4.  **Summary**: Similar to Histogram, but calculates configurable quantiles on the client side.

### 3. Architecture
*   **Prometheus Server**: Scrapes and stores time series data.
*   **Exporters**: Libraries that export existing metrics from third-party systems (e.g., `node_exporter` for Linux metrics, `postgres_exporter`).
*   **Pushgateway**: For short-lived jobs that cannot be scraped.
*   **Alertmanager**: Handles alerts sent by Prometheus server (deduplication, grouping, routing to Slack/Email).
*   **PromQL**: A powerful functional query language to select and aggregate time series data in real time.

### 4. Pull vs. Push Model
Prometheus primarily uses a **Pull Model**.
*   The server scrapes metrics from targets at regular intervals (Scrape Interval).
*   Targets expose a `/metrics` endpoint in plain text.

## Go Ecosystem

### Using `client_golang`
**Evidence** ([source](https://github.com/prometheus/client_golang/blob/main/prometheus/promauto/auto.go)):
```go
import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
    "net/http"
)

var (
    opsProcessed = promauto.NewCounter(prometheus.CounterOpts{
        Name: "myapp_processed_ops_total",
        Help: "The total number of processed events",
    })
)

func main() {
    // Record a metric
    opsProcessed.Inc()

    // Expose /metrics
    http.Handle("/metrics", promhttp.Handler())
    http.ListenAndServe(":2112", nil)
}
```

## Interview Preparation Questions

**Q1: What is a "Rate" in PromQL and why is it important?**
**A1:** `rate(http_requests_total[5m])` calculates the per-second average rate of increase of the time series in the range vector (last 5 mins). It is crucial for visualizing throughput spikes and trends.

**Q2: How do you handle Prometheus scalability?**
**A2:** Prometheus is designed to be single-node. For long-term storage and global querying, we use projects like **Thanos** or **Cortex** which provide a distributed, long-term storage backend for Prometheus.

**Q3: What is "Service Discovery" in Prometheus?**
**A3:** Prometheus can automatically discover targets via integrations with Kubernetes, AWS, etc. This is essential in dynamic environments where IP addresses of pods/instances change constantly.
