---
---

# 03. Performance Monitoring

## Summary
Performance monitoring focuses on the speed and efficiency of the system. In modern architecture, this moves from tracking "Average Latency" to analyzing the **Tail Latency** (p99) and resource consumption using APM tools.

## Detailed Development

### 1. Latency: Average vs. Tail
Average latency is often a misleading metric because it hides the experience of the unluckiest users.
*   **The Problem**: If 99 requests take 10ms and 1 request takes 10s, the average is ~110ms. This looks "fine" but 1% of users are having a terrible experience.
*   **The Solution**: Use **Percentiles**.
    *   **p50 (Median)**: What half of the users experience.
    *   **p95/p99**: The "tail" of the distribution. These usually reveal bottlenecks in DB locks, garbage collection (GC) pauses, or network congestion.
*   **Histograms**: Data structure used to collect latency data into "buckets" (e.g., [0-50ms], [50-100ms]).

### 2. APM (Application Performance Monitoring)
APM provides deep visibility into the code execution.
*   **Auto-instrumentation**: Agents that hook into the runtime to measure function call duration automatically.
*   **Service Maps**: Visualizing how services call each other and where the delay is happening.
*   **Flame Graphs**: Visualizing CPU profiles to find "hot" functions that consume the most resources.

### 3. Key Metrics to Watch
*   **Response Time**: Total time to complete a request.
*   **Throughput**: Requests per second (RPS).
*   **Resource Utilization**: CPU, Memory, Disk I/O, Network Bandwidth.
*   **Database Performance**: Query execution time, connection pool saturation.

## Go Ecosystem
Go provides powerful built-in tools for performance monitoring, specifically `pprof` and `expvar`.

### 1. Prometheus Histograms in Go
**Evidence** ([source](https://github.com/milvus-io/milvus/blob/master/pkg/metrics/proxy_metrics.go#L67)):
```go
var ProxySQLatency = prometheus.NewHistogramVec(
    prometheus.HistogramOpts{
        Namespace: "api",
        Name:      "request_duration_seconds",
        Help:      "Latency of search or query successfully",
        Buckets:   prometheus.DefBuckets, // default: .005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10
    }, []string{"method", "status"})
```

### 2. Live Profiling with pprof
Go can expose a web interface for real-time profiling.
```go
import _ "net/http/pprof"

func main() {
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    // ... app logic
}
```
You can then run `go tool pprof http://localhost:6060/debug/pprof/profile` to capture a 30s CPU profile.

## Interview Preparation Questions

**Q1: Why do we use p99 instead of p100 (Max)?**
**A1:** p100 (the maximum value) is often an outlier caused by a single, non-repeatable event (e.g., a one-time network glitch). p99 is more stable and represents a real trend of the slowest legitimate requests.

**Q2: How does a slow backend affect a frontend's p99?**
**A2:** Due to the "Amplification Effect," if a frontend calls 10 backends, and each has a 1% chance of being slow (p99), the frontend has an $\approx 10\%$ chance of being slow. This is why backend tail latency is critical.

**Q3: Explain the performance overhead of an APM agent.**
**A3:** APM agents add overhead by intercepting calls and generating telemetry. This typically ranges from 1% to 5% CPU/Memory overhead. In high-performance systems, we use **Sampling** (only tracing 1% of requests) to minimize this.
