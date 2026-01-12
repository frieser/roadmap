---
---

# 19. Observability

## Summary
Observability is a measure of how well internal states of a system can be inferred from knowledge of its external outputs. In modern backend engineering, it moves beyond traditional monitoring (**Known Unknowns**) to enable deep exploration of system behavior (**Unknown Unknowns**).

## Detailed Development

### 1. Monitoring vs Observability
- **Monitoring (Known Unknowns)**: Focuses on tracking predefined metrics and health checks. It tells you *that* something is wrong (e.g., "CPU is high").
- **Observability (Unknown Unknowns)**: Provides the context to understand *why* something is wrong. It allows you to ask questions you didn't anticipate when building the system.

### 2. The Three Pillars of Observability

#### A. Metrics (Aggregatable Data)
Numerical measurements recorded over time.
- **Counters**: Monotonically increasing (e.g., total requests).
- **Gauges**: Values that go up and down (e.g., memory usage).
- **Histograms**: Distribution of values (e.g., request latency).
- **Cardinality Issue**: High cardinality occurs when a label has too many unique values (e.g., `user_id`, `request_id`). This leads to a combinatorial explosion of time series, causing memory issues in TSDBs like Prometheus.

#### B. Logs (Event Data)
Immutable, timestamped records of discrete events.
- **Structured Logs**: JSON format (e.g., Zap, Zerolog) allows for easy querying and filtering by machines.
- **Sampling Strategies**: To reduce storage costs, systems may use **Head-based sampling** (sampling at the start) or only log errors/warnings in production.

#### C. Traces (Distributed Request Context)
Tracks a single request journey across service boundaries.
- **Spans**: The basic building block, representing a single operation.
- **Context Propagation**: The mechanism of passing `TraceID` and `SpanID` through service calls (e.g., via W3C `traceparent` headers).

### 3. Telemetry Models
- **Pull Model**: The observability backend (Prometheus) scrapes data from the application. Better for service discovery and preventing "DDoS by telemetry."
- **Push Model**: The application pushes data to a collector (OTLP, Jaeger). Essential for short-lived tasks (Serverless/Lambda).
- **OpenTelemetry (OTel)**: A vendor-neutral framework. The **OTel Collector** acts as a buffer/processor between your app and various backends (Grafana, Datadog, Jaeger).

### 4. Instrumentation
- **Manual**: Explicitly adding code to create spans and record metrics.
- **Auto-instrumentation (eBPF)**: Uses kernel-level hooks to capture network and system calls without changing application code. Low overhead but often less context than manual instrumentation.

## Go Ecosystem

### Metrics with Prometheus
Go applications typically use the `prometheus/client_golang` library.

**Evidence** ([source](https://github.com/milvus-io/milvus/blob/master/pkg/metrics/proxy_metrics.go#L26)):
```go
var ProxyReceivedNQ = prometheus.NewCounterVec(
    prometheus.CounterOpts{
        Namespace: "milvus",
        Subsystem: "proxy",
        Name:      "received_nq",
        Help:      "counter of nq of received search and query requests",
    }, []string{"nodeID", "queryType", "database", "collectionName"})
```

### Tracing with OpenTelemetry
Using `opentelemetry-go` for distributed tracing.

**Evidence** ([source](https://github.com/milvus-io/milvus/blob/master/internal/proxy/impl.go#L111)):
```go
ctx, sp := otel.Tracer("ProxyRole").Start(ctx, "Proxy-InvalidateCollectionMetaCache")
defer sp.End()
```

### High-Performance Logging
`uber-go/zap` or `rs/zerolog` are preferred for zero-allocation structured logging.
```go
logger, _ := zap.NewProduction()
defer logger.Sync()
logger.Info("failed to fetch URL",
  zap.String("url", url),
  zap.Int("attempt", 3),
  zap.Duration("backoff", time.Second),
)
```

## Go-specific Applications
- **Context Propagation**: In Go, `context.Context` is the vehicle for trace propagation. Always pass `ctx` to downstream functions and external calls.
- **Middleware**: Use `otelhttp` or `otellogrus` middleware to automatically wrap handlers with spans and log correlation.

## Interview Preparation Questions

**Q1: What is High Cardinality in Prometheus and why is it dangerous?**
**A1:** High cardinality happens when you use labels with many unique values (like `email` or `user_id`). Prometheus creates a new time series for every unique combination of labels. This causes "TSDB Bloat," leading to high memory usage and slow query performance.

**Q2: Compare Tail-based sampling vs Head-based sampling.**
**A2:**
- **Head-based**: Decisions are made at the start of a trace. Simple but might miss rare errors if the sample rate is low.
- **Tail-based**: The decision is made *after* the entire trace is finished (usually in an OTel Collector). This allows for "100% of errors" to be kept while discarding 99% of successful traces.

**Q3: How does Context Propagation work in a Go microservice?**
**A3:** It involves **Injecting** the trace context into outbound headers (e.g., HTTP `traceparent`) and **Extracting** it in the receiving service. The `otel` SDK uses `TextMapPropagator` for this. The `context.Context` object must be passed through every function call to maintain the span hierarchy.

