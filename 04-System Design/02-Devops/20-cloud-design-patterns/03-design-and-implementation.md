---
---

# Cloud Design Patterns: Design & Implementation

These patterns help in designing the application architecture for maintainability and reusability.

## Summary

*   **Sidecar**: Deploys components of an application into a separate process or container to provide isolation and encapsulation (e.g., logging, proxying).
*   **Ambassador**: Creates a helper service that sends network requests on behalf of a consumer service or application (e.g., smart client).
*   **Strangler Fig**: Incrementally migrates a legacy system by gradually replacing specific pieces of functionality with new applications and services.

---

## Go Implementation: Sidecar Pattern

In Go, a Sidecar is often implemented as a separate Goroutine or a separate binary in the same Pod. This example shows a main app and a "Sidecar" logger running concurrently.

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

// Main Application
func mainApp() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// Log to the sidecar (via localhost)
		go logToSidecar("Request received")
		w.Write([]byte("Hello from Main App"))
	})
	log.Fatal(http.ListenAndServe(":8080", nil))
}

// Sidecar Logic (Simulation)
func sidecar() {
	http.HandleFunc("/log", func(w http.ResponseWriter, r *http.Request) {
		// Sidecar handles log shipping/processing logic
		fmt.Println("[Sidecar] Processing log entry...")
	})
	log.Fatal(http.ListenAndServe(":9090", nil))
}

func logToSidecar(msg string) {
	// Fire and forget log to localhost sidecar
	http.Get("http://localhost:9090/log?msg=" + msg)
}

func main() {
	// Start Sidecar
	go sidecar()
	
	// Wait for sidecar to boot
	time.Sleep(1 * time.Second)

	// Start Main App
	fmt.Println("Starting Main App...")
	mainApp()
}
```

## Interview Questions

**Q: How does the Strangler Fig pattern minimize risk during migration?**
**A:** Instead of a "Big Bang" rewrite (shut down old, start new), Strangler Fig places a proxy (like Nginx) in front of the legacy system. You rewrite *one* endpoint (e.g., `/api/users`) in the new system and route traffic for *only* that endpoint to the new app. Everything else goes to legacy. If it fails, you just revert the route. You repeat this until the legacy system is "strangled" and can be decommissioned.

**Q: What is the difference between Sidecar and Ambassador?**
**A:**
*   **Sidecar**: Enhances the application (Logging, Config, mTLS). Generic helper.
*   **Ambassador**: Specifically handles *outbound* network connectivity. It acts as a smart proxy for the application to talk to the outside world (e.g., the app talks to `localhost`, and the Ambassador handles Circuit Breaking, Retries, and Service Discovery to find the real remote service).
