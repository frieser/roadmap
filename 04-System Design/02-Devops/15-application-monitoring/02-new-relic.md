---
---

# New Relic

New Relic is a comprehensive observability platform. It was one of the first to offer SaaS-based APM and remains a strong contender for enterprise-grade monitoring.

## Summary

New Relic provides deep code-level visibility. It uses an **Agent-based** model where language-specific agents (Go, Java, Python) run inside the application process. It excels at diagnosing performance bottlenecks like slow database queries, external service calls, and runtime errors.

## Detailed Explanation

### 1. Key Features
*   **Transaction Tracing**: Identifies the exact line of code or SQL query causing slowness.
*   **Apdex Score**: An industry-standard metric for user satisfaction based on response time thresholds.
*   **Errors Inbox**: A dedicated view for aggregating and triaging runtime exceptions.

### 2. Go Agent
The New Relic Go Agent (`go-agent`) requires manual instrumentation (unlike Java/Ruby which can auto-instrument via bytecode injection). You must wrap your HTTP handlers and manually create "Transactions".

---

## Go Implementation Example

```go
package main

import (
	"fmt"
	"net/http"
	"os"
	"time"

	"github.com/newrelic/go-agent/v3/newrelic"
)

func main() {
	// 1. Create Application
	app, err := newrelic.NewApplication(
		newrelic.ConfigAppName("My Go App"),
		newrelic.ConfigLicense(os.Getenv("NEW_RELIC_LICENSE_KEY")),
		newrelic.ConfigAppLogForwardingEnabled(true),
	)
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	// 2. Instrument Handler
	http.HandleFunc(newrelic.WrapHandleFunc(app, "/hello", func(w http.ResponseWriter, r *http.Request) {
		// Manual Segment (e.g., complex calculation)
		txn := newrelic.FromContext(r.Context())
		defer txn.StartSegment("calculation").End()

		time.Sleep(100 * time.Millisecond)
		w.Write([]byte("Hello New Relic!"))
	}))

	http.ListenAndServe(":8000", nil)
}
```

## Interview Questions

**Q: What is Apdex and how is it calculated?**
**A:** Apdex (Application Performance Index) is a score from 0 to 1 measuring user satisfaction.
*   **Satisfied (T)**: Fast response.
*   **Tolerating (4T)**: Slow but usable.
*   **Frustrated (>4T or Error)**: Too slow or failed.
*   *Formula*: `(Satisfied + (Tolerating / 2)) / Total Samples`.

**Q: Why does the Go agent require more manual code than the Java agent?**
**A:** Java runs on a Virtual Machine (JVM), allowing agents to inject bytecode at runtime to intercept method calls automatically. Go compiles to native machine code. There is no VM to hook into, so developers must explicitly import the agent library and wrap their functions to capture telemetry.

**Q: What is a "Transaction" in New Relic?**
**A:** A Transaction represents a logical unit of work in an application, such as an HTTP request or a background job. It is the primary container for timing data, errors, and traced segments.
