---
---

## Summary
Server-Sent Events (SSE) is a push technology that enables a server to send automatic updates to a client via a single, long-lived HTTP connection. Unlike WebSockets, SSE is unidirectional (Server → Client) and is sent over standard HTTP, making it simpler to implement for scenarios like news feeds, stock tickers, or notification streams.

## Detailed Explanation

### How It Works
1.  **Client Request**: The client initiates a standard HTTP request to the server.
2.  **Server Response**: The server keeps the connection open and sends a response with the MIME type `text/event-stream`.
3.  **Streaming**: The server pushes text-based messages formatted in a specific way as they become available.
4.  **Auto-Reconnect**: The browser's `EventSource` API automatically handles connection drops and reattempts to connect.

### Message Format
SSE messages are text blocks separated by newlines.
```text
data: This is a message\n\n

event: update
data: {"status": "processing"}\n\n
```

### SSE vs WebSockets
| Feature | Server-Sent Events (SSE) | WebSockets |
| :--- | :--- | :--- |
| **Direction** | One-way (Server to Client) | Bidirectional (Full-duplex) |
| **Protocol** | Standard HTTP | WebSocket Protocol (over TCP) |
| **Complexity** | Low (Text-based, standard HTTP) | Medium (Binary framing, handshake) |
| **Reconnection** | Built-in (via `EventSource`) | Manual handling required |
| **Binary Data** | No (Text/Base64 only) | Yes |

## Go-Specific Context/Examples

In Go, SSE is implemented by keeping the HTTP handler running and flushing the response writer buffer periodically.

### Example: SSE Handler in Go

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func sseHandler(w http.ResponseWriter, r *http.Request) {
	// Set headers for SSE
	w.Header().Set("Content-Type", "text/event-stream")
	w.Header().Set("Cache-Control", "no-cache")
	w.Header().Set("Connection", "keep-alive")
	w.Header().Set("Access-Control-Allow-Origin", "*")

	// Create a channel to simulate data updates
	msgChan := make(chan string)

	// Simulate events in a goroutine
	go func() {
		for i := 0; i < 5; i++ {
			msgChan <- fmt.Sprintf("Time: %s", time.Now().Format(time.TimeOnly))
			time.Sleep(2 * time.Second)
		}
		close(msgChan)
	}()

	// Flush interface to send data immediately
	flusher, ok := w.(http.Flusher)
	if !ok {
		http.Error(w, "Streaming unsupported!", http.StatusInternalServerError)
		return
	}

	// Loop to send events
	for msg := range msgChan {
		// Format: "data: <message>\n\n"
		fmt.Fprintf(w, "data: %s\n\n", msg)
		flusher.Flush() // Push to client immediately
	}
	
	// Keep connection open until client disconnects or logic ends
	<-r.Context().Done()
	fmt.Println("Client disconnected")
}

func main() {
	http.HandleFunc("/events", sseHandler)
	fmt.Println("SSE Server running on :8080/events")
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: When would you choose SSE over WebSockets?**
**A:** Use SSE when you only need unidirectional communication (Server -> Client), such as stock tickers, news feeds, or status updates. It is simpler, works over standard HTTP (firewall-friendly), and has built-in reconnection logic. Use WebSockets if you need real-time bi-directional communication, like a chat app or multiplayer game.

**Q: Does SSE support binary data?**
**A:** No, SSE is strictly text-based (`text/event-stream`). To send binary data, you must encode it (e.g., Base64), which adds overhead. WebSockets support binary data natively.

**Q: How does a Go server know when an SSE client disconnects?**
**A:** The `http.Request` context (`r.Context()`) is canceled. You can listen for `<-r.Context().Done()` in your handler loop to clean up resources and stop sending data.
