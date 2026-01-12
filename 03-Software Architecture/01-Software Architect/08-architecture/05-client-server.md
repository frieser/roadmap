# Client-Server Architecture

## Summary
The Client-Server architecture is the foundational distributed system model where tasks are partitioned between service providers (**Servers**) and service requesters (**Clients**). It has evolved from simple 2-tier setups to complex N-tier distributed systems, adapting to modern needs through various communication protocols (REST, GraphQL, WebSocket) and state management strategies.

## Detailed Explanation

### 1. Evolution of Tiers

*   **2-Tier (Client + DB)**: Logic resides on the client (Fat Client) which talks directly to the database. Hard to maintain, insecure, and hard to scale.
*   **3-Tier (Client + Server + DB)**: Introduces a middleware/application server. Logic moves to the server. Secure, scalable, and manageable. The standard for web apps.
*   **N-Tier (Distributed)**: The server layer is broken down into multiple tiers (e.g., Microservices, Web Servers, App Servers, Caching Layers). Optimizes for specific concerns like load balancing and fault tolerance.

### 2. Client Types
*   **Thin Client**: Relies heavily on the server for processing. The client is mainly a display terminal (e.g., Early HTML websites, Citrix). Easier to manage updates.
*   **Thick (Fat) Client**: Performs significant local processing (e.g., SPAs like React, Mobile Apps, Desktop Games). Can work offline but requires more client resources and update management.

### 3. Communication Styles

| Style | Description | Best For |
| :--- | :--- | :--- |
| **REST** (Stateless) | Resource-based, standard HTTP methods. Cacheable. | Public APIs, simple CRUD, wide compatibility. |
| **GraphQL** (Query) | Client requests exactly what it needs. Single endpoint. | Mobile apps (bandwidth constrained), complex nested data. |
| **RPC** (Action) | Calling a remote function as if local (gRPC). | Microservices internal comms, high performance. |
| **WebSocket** (Duplex) | Persistent, bidirectional connection. | Real-time chat, gaming, live feeds. |

### 4. Stateless vs. Stateful Servers
*   **Stateless**: The server retains no session information between requests. Every request must contain all necessary info (e.g., JWT). **Essential for horizontal scaling**.
*   **Stateful**: The server keeps track of client sessions in memory. Simple to implement but makes scaling hard (requires Sticky Sessions).

## Go Code Example (REST Server)

```go
package main

import (
	"encoding/json"
	"net/http"
)

type Response struct {
	Message string `json:"message"`
}

func handler(w http.ResponseWriter, r *http.Request) {
	// Stateless: We don't check any server-side session variable.
	// We might check a token in the header here.
	
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(Response{Message: "Hello from Stateless Go Server"})
}

func main() {
	http.HandleFunc("/api/hello", handler)
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

*   **Q: Why is "Statelessness" preferred for modern Client-Server architectures?**
    *   **A:** It allows any server in a cluster to handle any request. If a server crashes, the client simply retries against another server without losing session data. It massively simplifies **Auto-scaling**.
*   **Q: When would you choose GraphQL over REST?**
    *   **A:** When the client needs to aggregate data from multiple resources (avoiding the "Under-fetching" / "N+1" problem of REST) or when bandwidth is expensive (mobile) and we want to avoid "Over-fetching" unnecessary fields.
*   **Q: What is the difference between a 3-Tier and an MVC architecture?**
    *   **A:** 3-Tier is a **physical/deployment** separation (Client, App Server, DB Server). MVC is a **logical/code** separation (Model, View, Controller) which often resides entirely within the "Presentation Tier" or the "Application Tier" of the 3-Tier system.
