---
---

# Avoid HTTP Blocking (Large Data)

In API design, blocking occurs when a request consumes server resources (threads, connections, memory) for an extended period, preventing other requests from being processed. This is particularly critical when dealing with large datasets or slow consumers.

## 1. Risks of Blocking

- **Slowloris Attacks**: A Denial of Service (DoS) attack where the attacker opens many connections to the server and keeps them open as long as possible by sending partial requests or headers very slowly. This exhausts the server's concurrent connection pool.
- **Thread/Worker Exhaustion**: Synchronous APIs that process large data in a single request-response cycle tie up worker threads. If all threads are busy generating or transmitting large payloads, the server becomes unresponsive to new traffic.
- **Timeout Issues**: Large data transfers are prone to timeouts at various layers (Client, Load Balancer, Gateway, or the Server itself).
    - **ReadTimeout**: Time spent waiting for the request body.
    - **WriteTimeout**: Time spent sending the response.
- **Memory Pressure**: Buffering large datasets in memory before sending them can lead to Out-Of-Memory (OOM) kills.

## 2. Solutions

### A. Pagination
Instead of returning all items, return a subset.
- **Offset-based**: Uses `page` and `limit`. Simple but inefficient for large offsets (`OFFSET 1000000` requires the DB to scan all previous rows).
- **Cursor-based (Keyset)**: Uses a pointer to the last element (`since_id`, `next_cursor`). More performant for deep pagination and stable against data shifts.

### B. Streaming
Send data to the client as it is generated, without buffering the entire set.
- **HTTP Chunked Encoding**: The server sends a `Transfer-Encoding: chunked` header and streams data in parts.
- **gRPC Streaming**: Highly efficient for service-to-service communication.
- **WebSockets**: Suitable for long-lived, bi-directional streams.

### C. Asynchronous Processing
For extremely large tasks (e.g., generating a 500MB PDF), offload the work.
1. Client submits a request.
2. Server returns **202 Accepted** with a `job_id`.
3. Background workers process the task.
4. Client retrieves the result via **polling** or **Webhooks**.

## 3. Go (Golang) Examples

### Streaming Large Data (HTTP)
Using `http.Flusher` to send data to the client incrementally.

```go
func streamHandler(w http.ResponseWriter, r *http.Request) {
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "text/plain")
    w.Header().Set("Transfer-Encoding", "chunked")

    for i := 1; i <= 10; i++ {
        fmt.Fprintf(w, "Chunk %d\n", i)
        flusher.Flush() // Send current buffer to client immediately
        time.Sleep(500 * time.Millisecond)
    }
}
```

### Pagination Logic (Cursor-based)
Example of a repository method using a cursor.

```go
func (r *Repo) GetUsers(cursor string, limit int) ([]User, string, error) {
    var users []User
    // SELECT * FROM users WHERE id > ? ORDER BY id ASC LIMIT ?
    err := r.db.Select(&users, "SELECT * FROM users WHERE id > ? LIMIT ?", cursor, limit)
    if err != nil {
        return nil, "", err
    }

    nextCursor := ""
    if len(users) > 0 {
        nextCursor = users[len(users)-1].ID
    }
    return users, nextCursor, nil
}
```

## 4. Interview Preparation

### Questions
1. **How do you protect a Go HTTP server from Slowloris attacks?**
   - *Answer*: Set aggressive `ReadTimeout`, `ReadHeaderTimeout`, and `IdleTimeout` in the `http.Server` configuration. Use a reverse proxy like Nginx or a Cloud WAF to buffer requests.
2. **What are the pros and cons of Cursor-based pagination?**
   - *Pros*: Performance (O(1) lookup), stability (no skipped items if data is added/deleted).
   - *Cons*: Cannot jump to a specific page, requires a unique/sequential column.
3. **When would you use gRPC streaming instead of a REST API?**
   - *Answer*: When transmitting large binary data, real-time updates between microservices, or when low latency and high throughput are required.
4. **Explain the "202 Accepted" pattern.**
   - *Answer*: It is an asynchronous pattern where the server acknowledges a long-running request without completing it. The client is given a way to track the status (polling or webhook).

### Key Terms
- **Slowloris**: Low-and-slow DoS attack.
- **Backpressure**: Telling a producer to slow down because the consumer cannot keep up.
- **Flusher**: Interface in Go to send buffered data to the client.
- **Idempotency**: Ensuring that the same operation can be repeated without changing the result (critical for async retries).
