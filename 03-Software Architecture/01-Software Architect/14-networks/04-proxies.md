# Proxies

Proxies are intermediary servers that sit between a client and a destination server. They intercept, process, and forward requests and responses to improve security, performance, and control.

## 1. Definitions

### Forward Proxy (Client-Side)
A **Forward Proxy** acts on behalf of the client (or a group of clients). It sits between the client and the public internet.
- **Goal**: Protect the client's identity and control access to the internet.
- **Visibility**: The destination server sees the proxy's IP address, not the client's.
- **Analogy**: A spokesperson who asks questions on your behalf so people don't know who you are.

### Reverse Proxy (Server-Side)
A **Reverse Proxy** acts on behalf of the server (or a group of servers). It sits between the internet and the backend servers.
- **Goal**: Protect the servers, balance load, and optimize performance.
- **Visibility**: The client thinks they are talking directly to the destination server, but they are actually talking to the proxy.
- **Analogy**: A receptionist at a large office who handles all visitors and directs them to the right person.

| Feature | Forward Proxy | Reverse Proxy |
| :--- | :--- | :--- |
| **Focus** | Client Privacy / Control | Server Scalability / Security |
| **Location** | Client's Network | Server's Network |
| **Common Use** | Bypass censorship, filter content | Load balancing, SSL termination |
| **Client Awareness**| Usually configured by client | Transparent to client |

---

## 2. Use Cases

- **Load Balancing**: Distributing incoming traffic across multiple backend servers to ensure high availability and prevent any single server from becoming a bottleneck.
- **Caching**: Storing copies of frequently requested content (e.g., static assets like images, CSS) to reduce latency and server load.
- **SSL Termination**: Decrypting incoming HTTPS requests at the proxy level and forwarding them as plain HTTP to backend servers, reducing the computational load on the application servers.
- **Anonymity & Privacy**: Hiding the client's IP (Forward Proxy) or the server's infrastructure details (Reverse Proxy).
- **Security & Filtering**: Blocking malicious traffic, preventing DDoS attacks, and enforcing access control policies (e.g., blocking social media at work).
- **Rate Limiting**: Controlling the number of requests a client can make in a given timeframe.

---

## 3. Technologies

### Nginx
The most popular open-source web server that also functions as a powerful reverse proxy and load balancer. Known for its event-driven architecture and high performance.

### HAProxy (High Availability Proxy)
A specialized, high-performance TCP/HTTP load balancer. It is often chosen for its advanced load-balancing algorithms and deep health check capabilities at Layer 4 and Layer 7.

### Envoy
A modern, cloud-native "edge and service proxy" designed for microservices. It is a key component of service meshes like Istio, providing advanced observability and dynamic configuration.

---

## 4. Go Implementation: `httputil.ReverseProxy`

In Go, the standard library provides a robust implementation for building a reverse proxy through the `net/http/httputil` package.

### Concise Example
The following example demonstrates how to create a simple reverse proxy that forwards all requests to a target backend server.

```go
package main

import (
	"log"
	"net/http"
	"net/http/httputil"
	"net/url"
)

func main() {
	// 1. Define the backend server URL
	target := "http://localhost:8080"
	url, err := url.Parse(target)
	if err != nil {
		log.Fatal(err)
	}

	// 2. Create the Reverse Proxy
	proxy := httputil.NewSingleHostReverseProxy(url)

	// 3. Start the server
	log.Printf("Starting proxy on :9090 forwarding to %s", target)
	if err := http.ListenAndServe(":9090", proxy); err != nil {
		log.Fatal(err)
	}
}
```

### **Claim**: The `httputil.ReverseProxy` manages hop-by-hop headers and request rewriting internally.

**Evidence** ([source](https://github.com/golang/go/blob/28147b528312055b535c6a69d0d4492bd502e1b0/src/net/http/httputil/reverseproxy.go#L301-L315)):
```go
// NewSingleHostReverseProxy returns a new [ReverseProxy] that routes
// URLs to the scheme, host, and base path provided in target.
func NewSingleHostReverseProxy(target *url.URL) *ReverseProxy {
	director := func(req *http.Request) {
		rewriteRequestURL(req, target)
	}
	return &ReverseProxy{Director: director}
}

func rewriteRequestURL(req *http.Request, target *url.URL) {
    // Internal logic for path joining and query parameter handling
    // ...
}
```

**Explanation**: When using `NewSingleHostReverseProxy`, Go automatically creates a `Director` function that handles the URL transformation. Additionally, it strips "hop-by-hop" headers (like `Connection`, `Keep-Alive`, `Upgrade`) as required by RFC 7230 to ensure the proxy behaves correctly in a chain of servers.

---

## 5. Interview Questions

**Q1: What is the main difference between a forward proxy and a reverse proxy?**
*   **Answer**: A forward proxy acts on behalf of the **client** (protecting client identity/filtering outbound traffic), whereas a reverse proxy acts on behalf of the **server** (load balancing, SSL termination, protecting server infrastructure).

**Q2: What is SSL Termination, and why do we do it at the proxy?**
*   **Answer**: SSL Termination is the process of decrypting HTTPS traffic at the proxy level. We do it to offload the expensive cryptographic computations from the application servers, allowing them to focus on business logic and simplifying certificate management (one place to update certs).

**Q3: Explain the difference between Layer 4 and Layer 7 proxying.**
*   **Answer**: **Layer 4** (Transport Layer) proxies based on IP and Port without looking at the packet content (fast, simple). **Layer 7** (Application Layer) looks at the HTTP headers, URLs, and cookies to make routing decisions (slower but more intelligent, e.g., routing `/api` to a different service than `/static`).

**Q4: How does a reverse proxy improve system security?**
*   **Answer**: It hides the identity and internal IP addresses of backend servers, provides a single point for Web Application Firewalls (WAF), handles DDoS mitigation, and can enforce centralized authentication/authorization.

**Q5: When would you use Envoy instead of Nginx?**
*   **Answer**: Envoy is preferred in **cloud-native/microservices** environments where dynamic configuration (via xDS API), advanced observability (distributed tracing, metrics), and service mesh integration (like Istio) are required. Nginx is often better for traditional web serving and simpler static content delivery.
