---
---

# Datadog APM

Datadog APM (Application Performance Monitoring) provides end-to-end distributed tracing from the frontend to the database. It is renowned for its "Auto-Instrumentation" capabilities and ease of setup.

## Summary

Datadog APM uses a **Push-based** model where the application sends traces to a local **Datadog Agent** (via HTTP or UDS), which then forwards them to the cloud. Its Go library (`dd-trace-go`) includes wrappers for almost every popular Go framework (Gin, Echo, Chi, gRPC) and database driver (Postgres, MySQL, Redis), allowing for instant visibility with minimal code changes.

## Detailed Explanation

### 1. Traces, Spans, and Services
*   **Trace**: A request's entire journey through your system. Composed of multiple Spans.
*   **Span**: A single unit of work (e.g., "SQL Query", "HTTP Handler").
*   **Service**: A logical grouping of spans (e.g., `user-auth-service`).

### 2. Integration
*   **Automatic Injection**: Datadog injects trace IDs into HTTP headers (`x-datadog-trace-id`) to propagate context between microservices.
*   **Flame Graphs**: Visualizes the call stack and latency breakdown of a request.

---

## Go Implementation Example

Using `gopkg.in/DataDog/dd-trace-go.v1` to instrument an HTTP handler and a database call.

```go
package main

import (
	"database/sql"
	"log"
	"net/http"

	"github.com/lib/pq"
	sqltrace "gopkg.in/DataDog/dd-trace-go.v1/contrib/database/sql"
	httptrace "gopkg.in/DataDog/dd-trace-go.v1/contrib/net/http"
	"gopkg.in/DataDog/dd-trace-go.v1/ddtrace/tracer"
)

func main() {
	// 1. Start the Tracer
	tracer.Start(
		tracer.WithService("my-go-api"),
		tracer.WithEnv("production"),
	)
	defer tracer.Stop()

	// 2. Register Traced Database Driver
	sqltrace.Register("postgres", &pq.Driver{}, sqltrace.WithServiceName("my-db"))
	db, err := sqltrace.Open("postgres", "postgres://user:pass@localhost/db")
	if err != nil {
		log.Fatal(err)
	}

	// 3. Instrument HTTP Handler
	handler := func(w http.ResponseWriter, r *http.Request) {
		// The tracer automatically extracts context from headers
		ctx := r.Context()

		// Database call (automatically traced as a child span)
		var id int
		err := db.QueryRowContext(ctx, "SELECT id FROM users LIMIT 1").Scan(&id)
		if err != nil {
			http.Error(w, "DB Error", 500)
			return
		}
		
		w.Write([]byte("Request processed!"))
	}

	// Wrap the handler
	http.Handle("/users", httptrace.NewHandler(http.HandlerFunc(handler)))
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## Interview Questions

**Q: How does distributed tracing work across microservices?**
**A:** Distributed tracing relies on **Context Propagation**. When Service A calls Service B, it injects trace metadata (Trace ID, Parent Span ID) into the HTTP headers (e.g., `x-datadog-trace-id`). Service B extracts this metadata and uses the same Trace ID for its own spans, linking the two services in a single trace view.

**Q: What is the overhead of Datadog APM?**
**A:** The overhead is generally low (single-digit percentage CPU). The heavy lifting (aggregation, sampling) is offloaded to the Datadog Agent, which runs as a separate process. However, tracing every single request in a high-throughput system can be expensive, so **Sampling** is often used.

**Q: What is "Sampling" in APM?**
**A:** Sampling is the process of selecting only a subset of traces (e.g., 10%) to send to the backend to save on storage and processing costs. Datadog supports "Smart Sampling" (keeping interesting traces with errors or high latency) and explicit retention rules.
