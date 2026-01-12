---
---

## Summary
Long Polling is a technique used to emulate a server-push mechanism over standard HTTP. Unlike regular polling (where the client requests data at fixed intervals), in Long Polling, the client sends a request and the server **holds** the connection open until new data is available or a timeout occurs. Once data is received, the client immediately sends a new request, creating a near real-time loop.

## Detailed Explanation

### The Mechanism
1.  **Request**: Client sends an AJAX/HTTP request asking for new data.
2.  **Wait**: The server does *not* respond immediately if there is no new data. Instead, it hangs (sleeps/waits) until data arrives or a timeout (e.g., 30s) is reached.
3.  **Response**: Once data is available, the server returns a standard HTTP response (200 OK) with the payload.
4.  **Repeat**: The client receives the data and *immediately* issues a new request to start the process again.

### Pros and Cons
*   **Pros**: Works over standard HTTP, no special protocols (like WebSockets) needed, compatible with older browsers and strict firewalls.
*   **Cons**: Higher server resource usage (holding a thread/connection per client), header overhead for every message (unlike WebSockets), potential latency gap between the response and the next request.

### Long Polling vs WebSockets
Long polling was the standard before WebSockets. Today, it is largely a fallback mechanism (e.g., used by Socket.IO) when WebSockets are unavailable.

## Go-Specific Context/Examples

In Go, you can implement long polling using channels to block the handler until an event occurs or a context timeout triggers.

### Example: Long Polling Handler

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

// Message broker (simplified)
var messages = make(chan string)

func pollHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Println("Client connected, waiting for data...")

	// Use Context to handle client disconnects or timeouts
	ctx := r.Context()

	select {
	case msg := <-messages:
		// Data available: send it
		w.WriteHeader(http.StatusOK)
		w.Write([]byte(msg))
		fmt.Println("Sent message to client")
	case <-time.After(30 * time.Second):
		// Timeout: return empty response so client can reconnect
		w.WriteHeader(http.StatusNoContent) // 204 No Content
		fmt.Println("Timeout, no data")
	case <-ctx.Done():
		// Client disconnected
		fmt.Println("Client disconnected")
	}
}

func triggerHandler(w http.ResponseWriter, r *http.Request) {
	// Simulate an event
	go func() { messages <- "New Data Available at " + time.Now().String() }()
	w.Write([]byte("Event triggered"))
}

func main() {
	http.HandleFunc("/poll", pollHandler)
	http.HandleFunc("/trigger", triggerHandler)
	
	fmt.Println("Server running. Access /poll to wait, /trigger to send data.")
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: Why is Long Polling more resource-intensive than Short Polling for the server?**
**A:** Actually, it depends. Long polling keeps a connection open for every client, consuming file descriptors and potentially threads (depending on the server architecture). However, Short Polling consumes bandwidth and CPU by processing many empty requests. In modern async servers (like Go or Node), holding idle connections for Long Polling is generally efficient, but managing thousands of open TCP connections is the main bottleneck.

**Q: What happens if the client disconnects while the server is holding the Long Poll request?**
**A:** The server needs to detect this to free resources. In Go, the `http.Request` context (`r.Context().Done()`) will close. The handler must listen for this signal to stop waiting and return/exit, otherwise, the goroutine might leak.

**Q: How does Long Polling compare to WebSockets for battery life on mobile?**
**A:** WebSockets are generally better. Long Polling requires establishing a new TCP/TLS connection for every single message cycle (request -> response -> new request), which involves significant overhead and radio usage. WebSockets keep a single connection open, reducing the radio state changes.
