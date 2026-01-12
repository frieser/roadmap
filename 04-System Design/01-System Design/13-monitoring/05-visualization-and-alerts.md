---
---

# 05. Visualization and Alerts

## Summary
Visualization turns raw telemetry into human-understandable insights, while Alerts turn insights into action. Modern observability follows the "Symptom-based Alerting" philosophy to minimize noise and alert fatigue.

## Detailed Development

### 1. The Four Golden Signals (Google SRE)
If you can only monitor four things, monitor these:
1.  **Latency**: Time to service a request (p99).
2.  **Traffic**: Demand placed on the system (RPS).
3.  **Errors**: Rate of failed requests (5xx).
4.  **Saturation**: How "full" the system is (CPU/Mem/Thread pools).

### 2. Alerting Philosophy
*   **Symptom-based Alerting**: "Users are seeing errors" (Paging).
*   **Cause-based Alerting**: "CPU is 80%" (Non-paging ticket).
*   **Actionable Alerts**: Every alert must have a clear "Next Step" or Runbook link. If an alert doesn't require a human to do something, it shouldn't be an alert—it should be an automated script.
*   **Alert Fatigue**: Occurs when humans are paged for non-urgent or non-actionable issues. This leads to missing "real" pages.

### 3. Visualization Best Practices
*   **Grafana**: The industry-standard dashboarding tool.
*   **Consistency**: Use the same dashboard templates across all microservices (Red/Amber/Green status).
*   **Annotations**: Overlay "Deployment" or "Config Change" events on graphs to quickly correlate performance drops with changes.
*   **Drill-down**: Start with a "High-level" overview and allow clicking into specific service traces or logs.

## Go Ecosystem

### Instrumenting Golden Signals in Go
**Evidence** ([source](https://github.com/milvus-io/milvus/blob/master/pkg/metrics/proxy_metrics.go)):
```go
// 1. Errors: Counter for failures
var RequestErrors = prometheus.NewCounterVec(
    prometheus.CounterOpts{
        Name: "http_requests_errors_total",
        Help: "Total number of HTTP 5xx errors",
    }, []string{"endpoint"})

// 2. Traffic: Counter for total requests
var RequestCount = prometheus.NewCounterVec(
    prometheus.CounterOpts{
        Name: "http_requests_total",
        Help: "Total number of HTTP requests",
    }, []string{"endpoint"})

// 3. Latency: Histogram for p99
var RequestLatency = prometheus.NewHistogramVec(
    prometheus.HistogramOpts{
        Name: "http_request_duration_seconds",
    }, []string{"endpoint"})
```

### Saturation Monitoring
In Go, saturation often involves monitoring **Goroutine counts** and **Heap usage**.
```go
// Monitoring Goroutine saturation
numGoroutines := runtime.NumGoroutine()
if numGoroutines > threshold {
    // Alert: Possible goroutine leak
}
```

## Interview Preparation Questions

**Q1: What is the difference between a Dashboard and an Alert?**
**A1:** A Dashboard is for **investigation** and finding the "Why". An Alert is for **notification** that the "What" is broken. Dashboards are passive; Alerts are proactive.

**Q2: Explain "Error Budgeting" in SLOs.**
**A2:** If your SLO is 99.9% uptime, you have a 0.1% "Error Budget" (approx 43 mins/month). If you have budget left, you can deploy risky features. If the budget is exhausted, you freeze deployments and focus on stability.

**Q3: How do you prevent "Alert Storms"?**
**A3:** Use **Alert Grouping** and **Dependencies**. For example, if the Core Switch goes down, group the "Service Unreachable" alerts from 50 microservices into a single "Network Outage" alert.
