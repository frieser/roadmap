#API #Performance #Security
---
---

# Avoid HTTP Blocking (Large Data)

## Summary
Blocking HTTP requests to process large datasets or perform long-running tasks is a major anti-pattern. It leaves connections open for extended periods, making the server vulnerable to **Slowloris attacks**, **Thread Exhaustion**, and **Gateway Timeouts**. The solution is to use **Pagination** for large reads, **Streaming** for real-time data, and **Asynchronous Processing** (Job Queues) for heavy writes/computations.

## Detailed Explanation

### 1. Risks of HTTP Blocking
*   **Thread Exhaustion**: In thread-per-request models (like Java/Spring or older Apache), a slow request holds a thread. If 100 users request a large report, the server runs out of threads and stops responding to *everyone*.
*   **Timeouts**: Load balancers (Nginx/AWS ALB) typically have a 60-second timeout. If a request takes 61 seconds, the connection is cut, the user gets a 504 error, but the server continues processing the cancelled task (waste of resources).
*   **DoS**: Attackers can intentionally send large bodies slowly or request massive datasets to hold connections open (Resource Exhaustion).

### 2. Architectural Solutions
*   **Pagination**: Never return "All Users". Always use `limit` and `offset` (or Cursor-based pagination).
*   **Streaming**: Use **Chunked Transfer Encoding** to send data as it is generated, keeping the connection active and reducing memory usage.
*   **Asynchronous (202 Accepted)**:
    1.  Client sends `POST /report`.
    2.  Server returns `202 Accepted` with a `Location: /jobs/123`.
    3.  Server processes job in background (Worker Queue).
    4.  Client polls `/jobs/123` or waits for a Webhook.

---

## Go (Golang) Application

### 1. Streaming Response (Chunked)
Instead of buffering 1GB of data in memory, write it to the response writer as it's generated.

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func StreamHandler(w http.ResponseWriter, r *http.Request) {
	// Check if the writer supports flushing
	flusher, ok := w.(http.Flusher)
	if !ok {
		http.Error(w, "Streaming not supported", http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "text/event-stream")
	w.Header().Set("Cache-Control", "no-cache")

	for i := 0; i < 5; i++ {
		// Simulate expensive work
		time.Sleep(1 * time.Second)

		// Write chunk
		fmt.Fprintf(w, "Data chunk %d\n", i)

		// Flush immediately to client
		flusher.Flush()
	}
}
```

### 2. Keyset Pagination (Cursor)
Better than Offset pagination for performance and consistency.

```go
// GET /users?limit=10&after_id=1050
func ListUsers(w http.ResponseWriter, r *http.Request) {
    limit := 10
    afterID := r.URL.Query().Get("after_id")
    
    // SQL: SELECT * FROM users WHERE id > ? ORDER BY id ASC LIMIT ?
    users := db.QueryUsers(afterID, limit)
    
    json.NewEncoder(w).Encode(users)
}
```

---

## Interview Questions

**Q1: Why is Offset Pagination (`LIMIT 10 OFFSET 10000`) bad for performance?**
**A:** The database must read and discard 10,000 rows before returning the 10 requested. As the offset grows, the query gets slower (O(N)). **Cursor/Keyset Pagination** (`WHERE id > 10000 LIMIT 10`) allows the DB to jump directly to the correct index, maintaining O(1) or O(log N) performance.

**Q2: How do you handle a request that takes 5 minutes to complete (e.g., generating a PDF)?**
**A:** Do not block the HTTP request. Return **202 Accepted** immediately. Enqueue the task in a job queue (RabbitMQ/Redis). The client should either poll a status endpoint or listen for a Webhook notification when the task is done.

**Q3: What is a "Slowloris" attack?**
**A:** It's a DoS attack where the attacker opens many connections and sends partial HTTP requests (e.g., sending one header byte every 10 seconds). The server keeps the connection open waiting for the request to finish, eventually exhausting the maximum concurrent connection pool.

**Q4: In Go, how do you prevent a client from keeping a connection open forever?**
**A:** Configure `http.Server` timeouts: `ReadTimeout` (time to read body), `WriteTimeout` (time to write response), and `IdleTimeout`. Using middleware like `http.TimeoutHandler` can also enforce per-handler limits.
