---
---

## Summary
WebSockets provide a persistent, full-duplex communication channel between a client and a server over a single TCP connection. Unlike HTTP, which is request-response based, WebSockets allow both parties to send data independently at any time, making them ideal for high-frequency, low-latency real-time applications like chat, gaming, and collaborative tools.

## Detailed Explanation

### The Handshake
WebSockets start as a standard HTTP request. The client sends a request with an `Upgrade: websocket` header. If the server supports it, it responds with `101 Switching Protocols`, and the connection is upgraded from HTTP to the binary WebSocket protocol.

### Key Characteristics
1.  **Full-Duplex**: Client and Server can talk simultaneously.
2.  **Persistent Connection**: The connection stays open until explicitly closed, avoiding the overhead of opening a new TCP handshake for every message (unlike HTTP).
3.  **Low Overhead**: Messages (frames) have very small headers (as low as 2 bytes) compared to HTTP headers (cookies, user-agents, etc.).

### WebSockets vs HTTP
*   **HTTP**: Stateless, Request -> Response, Heavy headers. Good for documents and REST APIs.
*   **WebSocket**: Stateful connection, Event-driven, Lightweight framing. Good for real-time streams.

## Go-Specific Context/Examples

Go's standard library `net/http` does not support WebSockets out of the box. The community standard is `github.com/gorilla/websocket`, though `golang.org/x/net/websocket` exists (but is often considered less feature-rich).

### Example: Echo Server using Gorilla WebSocket

```go
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/gorilla/websocket"
)

// Upgrader promotes the HTTP connection to a WebSocket connection
var upgrader = websocket.Upgrader{
	ReadBufferSize:  1024,
	WriteBufferSize: 1024,
	// Allow all origins for example purposes
	CheckOrigin: func(r *http.Request) bool { return true },
}

func wsHandler(w http.ResponseWriter, r *http.Request) {
	// Upgrade initial GET request to a WebSocket
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println(err)
		return
	}
	defer conn.Close()

	fmt.Println("Client connected")

	for {
		// Read message from client
		messageType, p, err := conn.ReadMessage()
		if err != nil {
			log.Println("Read error:", err)
			break
		}
		fmt.Printf("Received: %s\n", p)

		// Echo message back to client
		if err := conn.WriteMessage(messageType, p); err != nil {
			log.Println("Write error:", err)
			break
		}
	}
}

func main() {
	http.HandleFunc("/ws", wsHandler)
	fmt.Println("WebSocket server started on :8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## Interview Questions

**Q: How does a WebSocket connection start?**
**A:** It starts with an HTTP "Handshake". The client sends a GET request with `Connection: Upgrade` and `Upgrade: websocket` headers. The server validates this and responds with status `101 Switching Protocols`, upgrading the TCP connection to the WebSocket protocol.

**Q: What are the scaling challenges with WebSockets compared to REST APIs?**
**A:** WebSockets are **stateful**. The server must maintain an open TCP connection for every active user, consuming file descriptors and memory (RAM). Load balancing is harder because you can't just round-robin requests; you often need "sticky sessions" or a pub/sub layer (like Redis) to broadcast messages across multiple server instances.

**Q: What is the purpose of the PING/PONG frames in WebSockets?**
**A:** They are heartbeat mechanisms to ensure the connection is still alive. Intermediaries (proxies, load balancers) often close idle connections; Ping/Pong frames keep traffic flowing to prevent these timeouts and detect dead clients.
