# Melody (WebSockets)

## Summary
Melody is a minimalist Go library for handling WebSocket connections. It is a wrapper around the popular (but low-level) `gorilla/websocket` package. Melody abstracts away the complexity of managing connection pools, concurrency, and message broadcasting, providing a simple, callback-based API similar to Socket.IO.

## Detailed Explanation

### 1. Key Features
*   **Broadcasting**: Send a message to all connected clients with `m.Broadcast()`.
*   **Concurrency**: Automatically handles concurrent writes (a common source of panics in raw WebSocket implementations).
*   **Sessions**: Associates data (like user ID) with a connection via `session.Set()`.

### 2. Usage Example
Melody integrates easily with `net/http` or frameworks like Gin.

```go
package main

import (
	"net/http"
	"github.com/olahol/melody"
)

func main() {
	m := melody.New()

	// 1. Handle HTTP Upgrade
	http.HandleFunc("/ws", func(w http.ResponseWriter, r *http.Request) {
		m.HandleRequest(w, r)
	})

	// 2. Define Callbacks
	m.HandleMessage(func(s *melody.Session, msg []byte) {
        // Echo message back to everyone (Chat room style)
		m.Broadcast(msg)
	})
    
    m.HandleConnect(func(s *melody.Session) {
        // New user connected
    })

	http.ListenAndServe(":5000", nil)
}
```

### 3. Session Management
You can store metadata on the connection object itself.
```go
m.HandleConnect(func(s *melody.Session) {
    s.Set("username", "Anonymous")
})
```

## Interview Questions

**Q: Why use Melody instead of raw `gorilla/websocket`?**
**A:** `gorilla/websocket` is a low-level implementation of the protocol. It is strict about concurrency: you cannot write to the same connection from multiple goroutines simultaneously without a mutex. Melody handles this internal locking and buffering for you, preventing "concurrent write to websocket connection" panics, and provides convenient `Broadcast` functionality out of the box.

**Q: Does Melody support "rooms" or "channels"?**
**A:** Not natively. Melody `Broadcast` sends to *all* connections. To implement rooms, you must filter sessions manually. `m.BroadcastFilter(msg, func(q *melody.Session) bool { return q.Get("room") == "general" })`.

**Q: Is Melody suitable for a distributed chat app (multiple server instances)?**
**A:** No. Melody stores sessions in memory on a single server. If you have 2 servers behind a load balancer, users on Server A cannot chat with users on Server B. For distributed real-time apps, you need a Pub/Sub backend (like Redis) or a dedicated service like **Centrifugo**.
