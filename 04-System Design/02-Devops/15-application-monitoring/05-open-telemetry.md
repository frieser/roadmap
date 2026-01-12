---
---

# OpenTelemetry (OTel)

OpenTelemetry is the industry standard for observability. It is a set of APIs, SDKs, and tools that allow you to generate, collect, and export telemetry data (metrics, logs, and traces) in a vendor-neutral way.

## Summary

Before OpenTelemetry, every vendor (Datadog, New Relic, Jaeger) had their own proprietary SDKs. Switching vendors required rewriting code. **OpenTelemetry** solves this by providing a single, standard way to instrument code. You instrument once with OTel, and then configure the "Exporter" to send data to any backend you choose.

## Detailed Explanation

### 1. The Three Pillars
*   **Traces**: The path of a request through services.
*   **Metrics**: Aggregated numerical data (CPU, request rates).
*   **Logs**: Timestamped text records.

### 2. Architecture
*   **OTel SDK**: The library inside your app that gathers data.
*   **OTLP (OpenTelemetry Protocol)**: The standard wire protocol for transmitting data.
*   **OTel Collector**: A standalone proxy. It receives data from apps (via OTLP), processes/filters it, and exports it to backends (e.g., sends traces to Jaeger, metrics to Prometheus).

---

## Go Implementation Example

Instrumenting a Go application with OpenTelemetry to generate Traces and Metrics.

```go
package main

import (
	"context"
	"log"
	"net/http"

	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/exporters/stdout/stdouttrace"
	"go.opentelemetry.io/otel/sdk/resource"
	sdktrace "go.opentelemetry.io/otel/sdk/trace"
	semconv "go.opentelemetry.io/otel/semconv/v1.17.0"
)

func initTracer() *sdktrace.TracerProvider {
	// For demo, export to STDOUT. In prod, use OTLP exporter.
	exporter, err := stdouttrace.New(stdouttrace.WithPrettyPrint())
	if err != nil {
		log.Fatal(err)
	}

	res, _ := resource.New(context.Background(), resource.WithAttributes(
		semconv.ServiceNameKey.String("go-otel-service"),
	))

	tp := sdktrace.NewTracerProvider(
		sdktrace.WithBatcher(exporter),
		sdktrace.WithResource(res),
	)
	otel.SetTracerProvider(tp)
	return tp
}

func main() {
	tp := initTracer()
	defer tp.Shutdown(context.Background())

	tr := otel.Tracer("main-component")

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// Start a Span
		ctx, span := tr.Start(r.Context(), "http-request")
		defer span.End()

		// Add custom attribute
		span.SetAttributes(attribute.String("user.id", "123"))

		w.Write([]byte("Hello OpenTelemetry!"))
	})

	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## Interview Questions

**Q: Why should I use OpenTelemetry instead of the Datadog/New Relic SDK directly?**
**A:** **Vendor Neutrality**. If you use the Datadog SDK, you are locked into Datadog. If you decide to switch to Honeycomb or AWS X-Ray later, you have to rewrite all your instrumentation code. With OpenTelemetry, you just change the configuration of the OTel Collector to export to the new vendor; the application code remains unchanged.

**Q: What is the OTel Collector?**
**A:** The Collector is a vendor-agnostic proxy that sits between your applications and your backend. It can receive telemetry data in multiple formats (OTLP, Jaeger, Prometheus), process it (filter PII, aggregate metrics, add metadata), and export it to multiple backends simultaneously (e.g., send metrics to Prometheus AND Datadog).

**Q: What is "W3C Trace Context"?**
**A:** It is the official web standard for trace propagation headers (`traceparent` and `tracestate`). OpenTelemetry uses this standard by default. This ensures that if your Go service calls a Java service (instrumented with OTel) or even an external vendor service, the trace context is preserved and understood universally without custom headers.
