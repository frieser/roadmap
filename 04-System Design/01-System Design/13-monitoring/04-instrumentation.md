---
---

# 04. Instrumentation

## Summary
Instrumentation is the act of adding code to your application to generate telemetry data. In 2026, the industry has converged on **OpenTelemetry (OTel)** as the standard for vendor-neutral instrumentation across the "Three Pillars of Observability": Logs, Metrics, and Traces.

## Detailed Development

### 1. The Three Pillars
*   **Metrics (Aggregatable)**: Numerical data (Counters, Gauges, Histograms). Best for alerting and real-time dashboards.
*   **Logs (Event-based)**: Timestamped records of discrete events. Crucial for understanding "What exactly happened?" during an error. Mandatory to use **Structured Logging** (JSON).
*   **Traces (Contextual)**: Follows a request's journey across service boundaries. Uses a `TraceID` to correlate events and a `SpanID` to identify specific operations.

### 2. Telemetry Models: Push vs. Pull
*   **Pull Model (Prometheus)**: The monitoring server "scrapes" an endpoint (e.g., `:8080/metrics`) exposed by the app.
    *   **Pros**: Service discovery is handled by the server; app doesn't need to know where the server is.
*   **Push Model (OTLP, Jaeger)**: The application "pushes" data to a collector or backend.
    *   **Pros**: Better for short-lived tasks (Lambda/Serverless) where there is no endpoint to scrape.

### 3. OpenTelemetry (OTel)
OTel provides a unified SDK to generate all three types of data.
*   **OTLP**: The protocol used to transport data.
*   **OTel Collector**: A middleman that receives data from apps, processes it (filtering/sampling), and exports it to multiple backends (e.g., Prometheus for metrics and Tempo for traces).

## Go Ecosystem

### 1. Structured Logging (Standard Library `slog`)
Go 1.21 introduced `log/slog` for high-performance structured logging.
```go
logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
logger.Info("processed request",
    slog.String("user_id", "123"),
    slog.Duration("latency", time.Since(start)),
)
```

### 2. Tracing with OpenTelemetry
**Evidence** ([source](https://github.com/milvus-io/milvus/blob/master/internal/proxy/impl.go#L111)):
```go
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
)

func (s *Handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Start a span
    ctx, span := otel.Tracer("my-service").Start(r.Context(), "ServeHTTP",
        trace.WithAttributes(attribute.String("http.method", r.Method)))
    defer span.End()

    // Context propagation is key in Go!
    doWork(ctx)
}
```

### 3. Context Propagation
In Go, `context.Context` is the vehicle for trace propagation. You must pass `ctx` through every function call to maintain the trace hierarchy.

## Interview Preparation Questions

**Q1: What is High Cardinality and why is it dangerous for Metrics?**
**A1:** High cardinality occurs when a label has many unique values (e.g., `user_id`, `request_id`). In TSDBs like Prometheus, every unique combination of labels creates a new time series. This leads to a memory explosion and crashes the monitoring system.

**Q2: Compare Manual vs. Auto-instrumentation (eBPF).**
**A2:**
*   **Manual**: Deep context, custom business metrics. Requires code changes.
*   **eBPF (Auto)**: Zero code changes, catches network/syscall issues. Lower context (doesn't know about user IDs unless explicitly passed).

**Q3: How do you correlate Logs and Traces?**
**A3:** By injecting the `TraceID` into every log line. Modern observability platforms (Grafana, Datadog) use this ID to allow "jumping" from a slow trace directly to the relevant log lines in one click.
