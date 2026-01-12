---
---

# Azure Functions

Azure Functions is Microsoft's serverless compute service. It stands out for its robust "Trigger and Binding" model, which drastically reduces the amount of boilerplate code developers need to write to connect to other Azure services (like CosmosDB or Storage Queues).

## Summary

Azure Functions supports Go via the **Custom Handler** pattern. Unlike C# or Node.js which have first-class worker integration, Go runs as a standalone executable that communicates with the Azure Functions Host via HTTP. This allows you to use standard Go HTTP patterns.

## Detailed Explanation

### 1. Triggers and Bindings
*   **Trigger**: What starts the function (HTTP, Timer, Queue Message).
*   **Binding**: Declarative way to connect data.
    *   *Input Binding*: "When triggered, fetch the document with ID=X from CosmosDB and pass it to my code."
    *   *Output Binding*: "Return this JSON, and Azure, please write it to this Blob Storage container."
    *   **Benefit**: Your code doesn't need to manage SDK connections or connection strings; the platform handles it.

### 2. Custom Handlers (The Go Way)
Since Go compiles to a binary, Azure treats it as a "Custom Handler."
1.  The Functions Host starts.
2.  It starts your Go web server.
3.  When an event occurs, the Host sends an HTTP request to your Go server.
4.  Your Go server processes it and returns a response.

---

## Go Implementation Example

### 1. The Configuration (`host.json`)
Tells Azure to forward requests to your executable.
```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "handler",
      "workingDirectory": "",
      "arguments": []
    },
    "enableForwardingHttpRequest": true
  }
}
```

### 2. The Go Handler (`main.go`)
It's just a standard HTTP server!

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
	// Standard Go HTTP handling
	name := r.URL.Query().Get("name")
	if name == "" {
		name = "Azure"
	}
	fmt.Fprintf(w, "Hello %s from Azure Functions Custom Handler!", name)
}

func main() {
	// Azure sets this environment variable
	port := os.Getenv("FUNCTIONS_CUSTOMHANDLER_PORT")
	if port == "" {
		port = "8080"
	}

	http.HandleFunc("/api/HttpTrigger1", helloHandler)

	log.Printf("Listening on port %s", port)
	log.Fatal(http.ListenAndServe(":"+port, nil))
}
```

## Interview Questions

**Q: What is a "Durable Function" in Azure?**
**A:** Durable Functions is an extension that lets you write *stateful* functions in a serverless environment. It allows you to define workflows (orchestrations) in code (e.g., "Run Function A, wait for it to finish, then run Function B and C in parallel, then aggregate results"). It manages the state checkpoints automatically in Azure Storage.

**Q: Explain the Consumption Plan vs. Premium Plan.**
**A:**
*   **Consumption Plan**: True serverless. You pay only when functions run. Scales to zero. Has cold starts and a timeout limit (default 5 mins).
*   **Premium Plan**: Provides pre-warmed instances (no cold starts), VNET connectivity, and longer run durations, but costs significantly more.

**Q: How does the "Custom Handler" model differ from AWS Lambda's Go runtime?**
**A:** In AWS Lambda, the Go runtime invokes your specific handler function directly via an RPC-like mechanism. In Azure's Custom Handler, your Go program runs as a long-lived **HTTP server**. The Azure Host proxies trigger events as standard HTTP requests to your web server. This makes the Go code very portable (it's just a web server).
