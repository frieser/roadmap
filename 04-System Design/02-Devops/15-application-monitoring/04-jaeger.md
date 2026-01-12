---
---

# Jaeger

Jaeger is an open-source, end-to-end distributed tracing system originally built by Uber. It is a CNCF graduated project used to monitor and troubleshoot transactions in complex distributed systems.

## Summary

Jaeger allows you to visualize the flow of a request as it passes through multiple microservices. It helps identify latency bottlenecks ("Why did this request take 3 seconds?") and dependency graphs. While it has its own SDKs, **OpenTelemetry** is now the recommended way to generate traces for Jaeger.

## Detailed Explanation

### 1. Architecture
*   **Agent**: Runs on the host. Listens for spans (UDP) and forwards them to the Collector.
*   **Collector**: Validates, transforms, and saves traces to storage.
*   **Storage**: Elasticsearch, Cassandra, or memory (for testing).
*   **Query/UI**: A React frontend to search and visualize traces.

### 2. The Move to OpenTelemetry
Jaeger's native client libraries (Jaeger Clients) have been deprecated. The modern approach is to use the **OpenTelemetry SDK** in your application and configure it to export traces to the Jaeger Collector (via OTLP).

---

## Go Implementation Example (Legacy vs Modern)

We will use the **Modern (OpenTelemetry)** approach, as the native Jaeger client is deprecated.

```go
package main

import (
	"context"
	"log"
	"net/http"

	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp"
	"go.opentelemetry.io/otel/sdk/resource"
	sdktrace "go.opentelemetry.io/otel/sdk/trace"
	semconv "go.opentelemetry.io/otel/semconv/v1.17.0"
)

func initTracer() (*sdktrace.TracerProvider, error) {
	ctx := context.Background()
	
	// Send traces to Jaeger (which accepts OTLP)
	exporter, err := otlptracehttp.New(ctx, otlptracehttp.WithEndpoint("jaeger-collector:4318"), otlptracehttp.WithInsecure())
	if err != nil {
		return nil, err
	}

	res, _ := resource.New(ctx, resource.WithAttributes(
		semconv.ServiceNameKey.String("order-service"),
	))

	tp := sdktrace.NewTracerProvider(
		sdktrace.WithBatcher(exporter),
		sdktrace.WithResource(res),
	)
	otel.SetTracerProvider(tp)
	return tp, nil
}

func main() {
	tp, _ := initTracer()
	defer tp.Shutdown(context.Background())

	tr := otel.Tracer("component-main")

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// Manually start a span
		_, span := tr.Start(r.Context(), "handle-request")
		defer span.End()

		w.Write([]byte("Trace sent to Jaeger"))
	})

	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## Interview Questions

**Q: What is a DAG in the context of Jaeger?**
**A:** Directed Acyclic Graph. Jaeger visualizes the dependencies between services as a DAG. If Service A calls B and C, and B calls D, the DAG shows the flow and hierarchy, helping architects understand system complexity and critical paths.

**Q: Why use UDP for the Jaeger Agent?**
**A:** The Jaeger Agent listens on UDP ports (e.g., 6831) for spans sent by the application. UDP is "fire and forget". This ensures that the tracing overhead never blocks the main application. If the agent is overloaded, it drops packets (traces) rather than slowing down the user request.

**Q: How does Context Propagation work?**
**A:** Context Propagation passes the Trace ID across process boundaries. In HTTP, this is done via headers (e.g., `traceparent` in W3C standard, or `uber-trace-id` in Jaeger native). The downstream service reads this header and creates a "Child Span" linked to the parent.
