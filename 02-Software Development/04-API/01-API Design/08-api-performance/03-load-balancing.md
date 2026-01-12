# Load Balancing

## Summary
Load Balancing is the process of distributing incoming network traffic across multiple servers to ensure high availability, reliability, and scalability. It prevents any single server from becoming a bottleneck. Load balancers can operate at Layer 4 (Transport) or Layer 7 (Application) and use algorithms like Round Robin, Least Connections, and IP Hash.

## Detailed Explanation

### Types of Load Balancing
1.  **Layer 4 (Transport Layer)**:
    *   Routes traffic based on IP address and port (TCP/UDP).
    *   Packet-level load balancing. Fast, but no insight into content.
    *   *Example*: AWS Network Load Balancer (NLB).

2.  **Layer 7 (Application Layer)**:
    *   Routes traffic based on content (URL path, headers, cookies).
    *   Slower (requires terminating SSL and inspecting packets), but smarter.
    *   *Example*: AWS Application Load Balancer (ALB), Nginx, HAProxy.

### Common Algorithms
*   **Round Robin**: Requests are distributed sequentially (Server A -> B -> C -> A). Simple, assumes equal server capacity.
*   **Weighted Round Robin**: Like RR, but servers with higher specs get more requests.
*   **Least Connections**: Sends traffic to the server with the fewest active connections. Good for long-lived connections (e.g., WebSockets).
*   **IP Hash**: Uses client IP to determine the server. Ensures a user always reaches the same server (Session Stickiness).

### Health Checks
Load balancers periodically check backend servers (e.g., `GET /health`). If a server fails, it is removed from rotation until it recovers.

### Go Implementation: Simple Reverse Proxy
Go's standard library `net/http/httputil` provides a powerful reverse proxy.

```go
package main

import (
	"log"
	"net/http"
	"net/http/httputil"
	"net/url"
	"sync/atomic"
)

// Simple Round Robin Load Balancer
type LoadBalancer struct {
	backends []*url.URL
	current  uint64
}

func (lb *LoadBalancer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	// 1. Select Backend (Round Robin)
	current := atomic.AddUint64(&lb.current, 1)
	backend := lb.backends[current%uint64(len(lb.backends))]

	log.Printf("Routing to %s", backend.String())

	// 2. Proxy Request
	proxy := httputil.NewSingleHostReverseProxy(backend)
	proxy.ErrorHandler = func(w http.ResponseWriter, r *http.Request, err error) {
		log.Printf("Error routing to %s: %v", backend, err)
		w.WriteHeader(http.StatusBadGateway)
	}
	proxy.ServeHTTP(w, r)
}

func main() {
	// Define backend servers
	urls := []string{
		"http://localhost:8081",
		"http://localhost:8082",
		"http://localhost:8083",
	}

	var backends []*url.URL
	for _, u := range urls {
		parsed, _ := url.Parse(u)
		backends = append(backends, parsed)
	}

	lb := &LoadBalancer{backends: backends}

	log.Println("Load Balancer running on :8080")
	log.Fatal(http.ListenAndServe(":8080", lb))
}
```

## Interview Questions

**Q: Explain the difference between L4 and L7 load balancing.**
**A:** L4 operates at the transport layer (TCP/UDP), making routing decisions based on IP and port without inspecting message content. It is faster but less flexible. L7 operates at the application layer (HTTP), allowing routing based on URL, headers, or cookies (e.g., routing `/api` to one service and `/images` to another).

**Q: How does a load balancer handle session persistence (sticky sessions)?**
**A:** It ensures requests from the same client always go to the same backend server. This is often implemented using **IP Hashing** (consistent hashing of client IP) or by injecting a **Session Cookie** that identifies the target server.

**Q: What happens when a health check fails?**
**A:** The load balancer marks the instance as "unhealthy" and stops routing new traffic to it. It continues to poll the instance; once health checks pass (often requiring multiple consecutive successes), it adds the instance back to the rotation.
