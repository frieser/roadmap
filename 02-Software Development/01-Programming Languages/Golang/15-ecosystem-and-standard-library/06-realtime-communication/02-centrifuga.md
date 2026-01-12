# Centrifugo

## Summary
Centrifugo is a scalable, real-time messaging server that acts as a language-agnostic broker for WebSocket (and other transport) connections. Unlike Melody (a library), Centrifugo is a standalone service (like Redis or NATS) that handles persistent connections, offering features like channel history, presence information, and automatic scalability, with a Go client library (`gocent`) for interaction.

## Detailed Explanation

### 1. Architecture
Centrifugo sits between your backend (Go, Python, PHP) and the frontend clients (Browsers, Mobile).
*   **Clients**: Connect to Centrifugo via WebSocket, SSE, or GRPC.
*   **Backend**: Publishes messages to Centrifugo via an API (HTTP or GRPC).
*   **Centrifugo**: Handles the "fan-out" to thousands of connected clients.

### 2. Key Features
*   **Scalability**: Nodes can be clustered using Redis or NATS as a "Broker Engine".
*   **History**: Can cache the last N messages in a channel for clients who momentarily disconnected.
*   **Presence**: Who is online in this channel?
*   **Recovery**: Automatic message recovery after disconnect.

### 3. Usage in Go
You typically run Centrifugo as a separate binary (Docker container). Your Go backend just publishes data.

```go
package main

import (
    "context"
    "github.com/centrifugal/gocent/v3"
)

func main() {
    // Connect to Centrifugo API
    c := gocent.New(gocent.Config{
        Addr: "http://localhost:8000/api",
        Key:  "my_api_key",
    })

    // Publish data to "news" channel
    ctx := context.Background()
    err := c.Publish(ctx, "news", []byte(`{"title": "Go is great"}`))
    if err != nil {
        panic(err)
    }
}
```

## Interview Questions

**Q: When should you use Centrifugo instead of a raw WebSocket library like Melody?**
**A:** Use Centrifugo when you need **scalability** (handling 100k+ connections across multiple server nodes), **persistence** (message history), or **advanced features** (user presence, automatic reconnection logic) out of the box. Melody is great for simple, single-server apps, but building a distributed cluster with it requires significant custom engineering.

**Q: How does Centrifugo handle authentication?**
**A:** It uses **JWT (JSON Web Tokens)**. When a client wants to connect to Centrifugo, it first asks your backend for a JWT. Your backend validates the user and signs a token containing the user ID and channel permissions. The client then passes this token to Centrifugo to establish the connection.

**Q: Can Centrifugo be embedded directly into a Go application?**
**A:** Yes, via the `centrifuge` library (which powers the Centrifugo server). This allows you to build a custom real-time server in Go with all the internal logic of Centrifugo, without running a separate process, though running it standalone is the most common pattern for polyglot microservices.
