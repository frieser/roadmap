---
---

# Cloud Design Patterns: Management & Monitoring

These patterns assist in the operations, administration, and monitoring of applications.

## Summary

*   **Bulkhead**: Isolates elements of an application into pools so that if one fails, the others will continue to function (like ship bulkheads).
*   **Scheduler Agent Supervisor**: Coordinates actions across a set of distributed actors.
*   **Retry**: Enables an application to handle transient failures when it tries to connect to a service or network resource.

---

## Go Implementation: Bulkhead Pattern

We use **Buffered Channels** as semaphores to limit the number of concurrent requests for specific resource types (e.g., Database vs External API).

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

// Bulkheads (Semaphores)
var dbLimit = make(chan struct{}, 5)  // Max 5 DB connections
var apiLimit = make(chan struct{}, 2) // Max 2 API calls

func DBHandler(w http.ResponseWriter, r *http.Request) {
	// Try to acquire semaphore (non-blocking or timeout)
	select {
	case dbLimit <- struct{}{}:
		defer func() { <-dbLimit }() // Release
		
		// Simulate DB work
		time.Sleep(100 * time.Millisecond)
		fmt.Fprintln(w, "DB Result")
		
	default:
		// Bulkhead full! Fail fast.
		http.Error(w, "Database Busy", http.StatusTooManyRequests)
	}
}

func APIHandler(w http.ResponseWriter, r *http.Request) {
	select {
	case apiLimit <- struct{}{}:
		defer func() { <-apiLimit }()
		
		// Simulate API work
		time.Sleep(500 * time.Millisecond)
		fmt.Fprintln(w, "API Result")
		
	default:
		http.Error(w, "API Busy", http.StatusTooManyRequests)
	}
}

func main() {
	http.HandleFunc("/db", DBHandler)
	http.HandleFunc("/api", APIHandler)
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: How does the Bulkhead pattern improve system resilience?**
**A:** Without bulkheads, a slow resource (e.g., a 3rd party API) can cause all threads/goroutines in your application to wait, eventually exhausting memory or file descriptors and crashing the *entire* application. Bulkheads ensure that if the "API" pool is full, new requests to the API fail immediately, but the "Database" pool remains free to serve other users. It prevents cascading failure.

**Q: When should you use the Retry pattern?**
**A:** Use Retry for **Transient Failures** (network blips, timeouts). **Do not** use it for permanent failures (404 Not Found, 401 Unauthorized) or internal application errors.

**Q: What is "Exponential Backoff"?**
**A:** When retrying, instead of retrying immediately (which might hammer a struggling server), you wait increasing amounts of time between attempts (e.g., 1s, 2s, 4s, 8s). This gives the downstream system time to recover.
