#Observability
---
---

## Summary
Telemetry refers to the data emitted by a system that describes its internal state and behavior. It is the fuel for Observability. The three primary pillars of telemetry are **Logs** (records of discrete events), **Metrics** (aggregated numerical data), and **Traces** (context of a request across services).

## Detailed Explanation

### The Three Pillars

1.  **Logs (Event-based)**
    *   *What*: "Something happened at time T."
    *   *Example*: `2023-10-01 12:00:01 [ERROR] User 123 failed login: Invalid password`.
    *   *Use*: Debugging specific errors, auditing, detailed root cause analysis.
    *   *Cost*: High (lots of storage text).

2.  **Metrics (Aggregatable)**
    *   *What*: "X happened Y times."
    *   *Example*: `login_failures_total = 52`.
    *   *Use*: Detecting trends, alerting, high-level dashboards.
    *   *Cost*: Low (constant storage regardless of traffic volume).

3.  **Traces (Request-scoped)**
    *   *What*: "Request A went from Service X -> Service Y -> DB."
    *   *Example*: A Span showing `Service A` took 50ms, then called `Service B` which took 200ms.
    *   *Use*: Understanding latency, visualizing dependencies, finding bottlenecks in microservices.

### OpenTelemetry (OTel)
OpenTelemetry is the industry standard (CNCF project) for generating and collecting this telemetry data. It provides a vendor-neutral SDK to instrument your code, so you can send data to Prometheus, Jaeger, Datadog, or Honeycomb without changing your code.

## Go-Specific Context/Examples

Using the OpenTelemetry Go SDK to create a Trace Span.

### Example: OTel Tracing in Go

```go
package main

import (
	"context"
	"log"
	"time"

	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/trace"
)

// Global tracer
var tracer trace.Tracer

func init() {
	// In a real app, you would configure a customized TracerProvider here
	// to export to Jaeger/Zipkin
	tracer = otel.Tracer("my-service-name")
}

func main() {
	ctx := context.Background()

	// Start a root span
	ctx, span := tracer.Start(ctx, "MainOperation")
	defer span.End()

	log.Println("Doing work...")
	doSubTask(ctx)
}

func doSubTask(ctx context.Context) {
	// Start a child span using the context from parent
	_, span := tracer.Start(ctx, "SubTask")
	defer span.End()

	// Simulate work
	time.Sleep(100 * time.Millisecond)
	
	// Add metadata to the span
	span.AddEvent("SubTask completed successfully")
}
```

## Interview Questions

**Q: When should you use Logs vs Metrics?**
**A:** Use **Metrics** for alerting and trending (e.g., "Alert if error rate > 1%"). Use **Logs** for investigation (e.g., "Show me the specific error message for User 123"). Metrics tell you *that* something is wrong; Logs tell you *what* specifically happened.

**Q: What is Distributed Tracing?**
**A:** Distributed tracing tracks a single request as it propagates through multiple microservices. It works by injecting a unique `TraceID` into the request headers (context propagation) so that logs and spans from different servers can be correlated together to reconstruct the full journey.

**Q: What is "Structured Logging"?**
**A:** Instead of logging raw text strings (`"User 123 logged in"`), you log JSON objects (`{"event": "login", "user_id": 123, "status": "success"}`). This allows log aggregation tools (like ELK or Splunk) to index fields efficiently, enabling powerful queries like `count(event) where status="failure"`. In Go, `slog` or `zap` are standard libraries for this.
