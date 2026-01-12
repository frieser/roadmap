---
---

## Summary
A **Load Balancer (LB)** and a **Reverse Proxy (RP)** are architecturally similar but serve different primary goals. A Load Balancer distributes incoming traffic across multiple servers to improve **scalability and availability**. A Reverse Proxy sits in front of one or more servers to handle **security, anonymity, and unified access** (masking the backend).

## Detailed Explanation

### Definitions
1.  **Load Balancer**: A device (or software) that acts as a traffic cop, sitting in front of your servers and routing client requests across all servers capable of fulfilling those requests. It maximizes speed and capacity utilization and ensures that no one server is overworked.
2.  **Reverse Proxy**: A type of proxy server that retrieves resources on behalf of a client from one or more servers. These resources are then returned to the client as if they originated from the proxy server itself.

### Key Differences

| Feature | Load Balancer | Reverse Proxy |
| :--- | :--- | :--- |
| **Primary Goal** | **Scale**: Distribute load. | **Control**: Security & Unification. |
| **Topology** | **One-to-Many**: One entry, many *identical* workers. | **One-to-One/Many**: One entry, specific *different* services. |
| **Health Checks** | Critical (removes bad nodes). | Optional (often just passes through). |
| **Focus** | Availability (uptime). | Anonymity (hiding internal IP). |

### The Overlap (NGINX / HAProxy)
Modern tools like NGINX and HAProxy function as **both**.
*   When NGINX routes `/api` to `Server A` and `/static` to `Server B`, it acts as a **Reverse Proxy**.
*   When NGINX routes `/api` to `Server A`, `Server B`, or `Server C` based on Round Robin, it acts as a **Load Balancer**.

## Go Example: Reverse Proxy

Using the standard library `httputil` to create a simple reverse proxy.

```go
package main

import (
	"log"
	"net/http"
	"net/http/httputil"
	"net/url"
)

func main() {
	// The backend server we are masking
	target, _ := url.Parse("http://localhost:8081")

	// Create the proxy
	proxy := httputil.NewSingleHostReverseProxy(target)

	// Modify the response (optional, e.g., to hide server headers)
	proxy.ModifyResponse = func(r *http.Response) error {
		r.Header.Del("Server")
		return nil
	}

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// Update headers to forward original IP
		r.Header.Set("X-Forwarded-Host", r.Header.Get("Host"))
		proxy.ServeHTTP(w, r)
	})

	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## Interview Questions

### Q: Can a Load Balancer act as a Reverse Proxy?
**A:** Yes. In fact, almost all Layer 7 Load Balancers act as Reverse Proxies by definition because they terminate the connection from the client and open a new one to the backend (acting as a proxy).

### Q: Why use a Reverse Proxy even if you have only one server?
**A:**
1.  **Security**: Shield the application server (e.g., Node.js/Go) from direct internet access.
2.  **SSL Termination**: Offload encryption/decryption overhead to the proxy (like NGINX).
3.  **Caching**: Serve static files directly from the proxy without hitting the app logic.
