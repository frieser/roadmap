# 17. Real-Time Data Communication

## Summary
Real-time data communication enables servers to push updates to clients with minimal latency. For senior backend developers, this involves choosing between **Polling**, **Long Polling (Comet)**, **Server-Sent Events (SSE)**, and **WebSockets** based on directionality, resource constraints (connection limits, memory), and scaling requirements (1M connections, Redis Pub/Sub).

## Detailed Explanation

### 1. Polling Strategies
#### Short Polling
The client sends periodic HTTP requests.
- **Pros**: Simple, stateless, works with all infrastructures.
- **Cons**: High overhead, "empty" requests waste resources.
- **Senior Consideration**: Use **Jitter** (randomized delay) to prevent "Thundering Herd" (simultaneous requests hitting the server) and **Exponential Backoff** to reduce load during failures.

#### Long Polling ("Comet")
The server holds the request until data is available or a timeout occurs.
- **Workflow**: Client Request $\to$ Server Waits $\to$ Data Arrives $\to$ Response $\to$ Client immediately re-requests.
- **Resource Exhaustion**: Each held connection consumes a thread/goroutine and memory. High churn of HTTP headers adds significant overhead compared to persistent sockets.

### 2. Server-Sent Events (SSE)
A standardized, uni-directional protocol (`text/event-stream`) over HTTP.
- **EventSource API**: Browser-native support for automatic reconnection and event IDs.
- **Connection Limits**:
    - **HTTP/1.1**: Browsers limit to **6 connections per domain**. This is a major bottleneck for multi-tab apps.
    - **HTTP/2**: Solves this via **multiplexing**, allowing many SSE streams over a single TCP connection.
- **Protocol**: Text-based, lightweight, but lacks binary support (requires Base64 encoding).

### 3. WebSockets (WS/WSS)
A full-duplex, persistent TCP connection that starts with an HTTP `101 Switching Protocols` handshake.
- **Framing**: Uses a small binary header (2-14 bytes).
- **Masking**: All frames from client $\to$ server MUST be masked to prevent malicious proxies from caching/poisoning sensitive data.
- **Stateful Scaling**: Since connections are persistent, the server is "stateful". Scaling requires a message broker like **Redis Pub/Sub** to broadcast messages across different server instances.

---

## Go Implementation

### 1. Polling with Jitter & Backoff
```go
func pollWithBackoff(ctx context.Context) {
    base := time.Second
    max := 30 * time.Second
    
    for i := 0; ; i++ {
        // Simulated work
        err := doPoll()
        if err == nil {
            i = 0 // Reset backoff on success
        }

        // Exponential Backoff: base * 2^i
        delay := float64(base) * math.Pow(2, float64(i))
        if delay > float64(max) {
            delay = float64(max)
        }

        // Add 10% Jitter
        jitter := (rand.Float64()*0.2 - 0.1) * delay
        sleepTime := time.Duration(delay + jitter)

        select {
        case <-time.After(sleepTime):
        case <-ctx.Done():
            return
        }
    }
}
```

### 2. SSE with `http.Flusher`
```go
func sseHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")

    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming unsupported", http.StatusInternalServerError)
        return
    }

    for {
        fmt.Fprintf(w, "data: %s\n\n", time.Now().Format(time.RFC3339))
        flusher.Flush()
        time.Sleep(time.Second)
    }
}
```

### 3. WebSockets: Gorilla vs Coder
| Feature | `gorilla/websocket` | `coder/websocket` |
| :--- | :--- | :--- |
| **API Style** | Traditional, Low-level | Modern, Idiomatic |
| **Context Support** | Poor (requires manual hacks) | First-class (`context.Context`) |
| **Concurrent Writes**| Not supported (needs Mutex) | Supported natively |
| **WASM Support** | No | Yes |

**Evidence** ([gorilla/websocket](https://github.com/gorilla/websocket/blob/e46b299/conn.go#L762)):
`gorilla` requires manual management of frame types and lacks native context support in its `ReadMessage` loop.

---

## Scaling to 1M Connections
To handle 1,000,000 concurrent connections in Go:
1.  **OS Limits**: Increase `ulimit -n` (file descriptors).
2.  **Memory**: 1M goroutines * 2KB stack = 2GB. Manageable, but idle connections should be offloaded.
3.  **Network Polling**: Use `epoll` (Linux) or `kqueue` (macOS/BSD). Libraries like `gnet` or `evio` bypass the "goroutine-per-connection" model to reduce context switching overhead for idle clients.
4.  **Pub/Sub**: Use Redis to sync state across nodes.

---

## Interview Questions

**Q: WS vs SSE for a high-frequency Stock Ticker?**
**A:** SSE is preferred. It is uni-directional (Server $\to$ Client), lighter on resources, works over standard HTTP/2, and handles reconnections natively. WebSockets are overkill unless the client needs to place trades (bi-directional).

**Q: Why do WebSockets use masking?**
**A:** To prevent **Cache Poisoning**. Old/broken proxies might see a WebSocket frame as a regular HTTP request and cache it. Masking ensures that every frame looks like "random" data to intermediate proxies.

**Q: How do you handle 1M connections in Go?**
**A:** Optimize `file descriptors`, use a non-blocking I/O library (epoll-based) to handle idle connections without spawning 1M goroutines, and use a distributed Pub/Sub (Redis) for message routing.

**Q: What is the "Thundering Herd" problem in polling?**
**A:** When many clients poll at the exact same interval (e.g., every 60s), they hit the server simultaneously. Adding **Jitter** to the interval spreads the load.
