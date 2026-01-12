---
---

# Nginx

Nginx (pronounced "engine-ex") is a high-performance HTTP server and reverse proxy. It is the de facto standard in modern DevOps for handling high-concurrency traffic, acting as an ingress controller in Kubernetes, and serving static content efficiently.

## Summary

Nginx uses an **asynchronous, event-driven architecture**, which allows it to handle thousands of concurrent connections with a small memory footprint (unlike Apache's thread-per-request model). Its primary use cases in DevOps include **Reverse Proxying** (routing traffic to backend apps), **Load Balancing** (distributing traffic), **TLS Termination**, and serving static files.

## Detailed Explanation

### 1. Architecture: Event-Driven
Instead of creating a new process or thread for every request, Nginx uses a master process and several worker processes.
*   **Worker Processes**: Run a non-blocking event loop (using `epoll` on Linux). One worker can handle thousands of connections by processing small chunks of work (reading request, sending response) and switching tasks while waiting for I/O.
*   **Scalability**: Scales linearly with CPU cores.

### 2. DevOps Use Cases
*   **Reverse Proxy**: Hides backend servers (like Go/Node.js apps) from the public internet.
*   **Ingress Controller**: The standard entry point for Kubernetes clusters (`ingress-nginx`), routing traffic based on hostnames (`api.example.com`) or paths (`/app`).
*   **API Gateway**: Rate limiting, authentication, and request routing.

### 3. Nginx Unit
A modern, dynamic application server from Nginx that can run code (Go, Python, PHP) and change configuration via REST API without reloading the process.

---

## Go Implementation Example

While Nginx is written in C, you can extend it or interact with it using Go.
1.  **Nginx Unit**: Control configuration programmatically.
2.  **Custom Modules**: Write Go plugins (via CGO).

### Interacting with Nginx Unit API
Nginx Unit allows you to reconfigure the server dynamically using JSON payloads sent to its control socket.

```go
package main

import (
	"bytes"
	"fmt"
	"net"
	"net/http"
	"os"
)

func main() {
	// Nginx Unit Control Socket
	socketPath := "/var/run/control.unit.sock"
	
	// Define a custom Transport to talk to the Unix socket
	transport := &http.Transport{
		Dial: func(proto, addr string) (net.Conn, error) {
			return net.Dial("unix", socketPath)
		},
	}
	client := &http.Client{Transport: transport}

	// JSON Configuration to deploy a Go app
	config := []byte(`{
		"listeners": {
			"*:8080": {
				"pass": "applications/my-go-app"
			}
		},
		"applications": {
			"my-go-app": {
				"type": "go",
				"executable": "/usr/bin/my-go-binary"
			}
		}
	}`)

	// Send PUT request to apply config
	// Note: Host header is required but ignored for Unix sockets
	req, _ := http.NewRequest("PUT", "http://localhost/config", bytes.NewBuffer(config))
	resp, err := client.Do(req)
	if err != nil {
		fmt.Printf("Error updating config: %v\n", err)
		os.Exit(1)
	}
	defer resp.Body.Close()

	fmt.Printf("Config updated: %s\n", resp.Status)
}
```

## Interview Questions

**Q: Explain the difference between Nginx and Apache's architecture.**
**A:** Apache (typically) uses a process-driven or thread-driven model (MPM Prefork/Worker), where each connection requires a dedicated thread/process. This consumes significant memory under high load (C10k problem). Nginx uses an asynchronous, event-driven architecture where a single worker handles thousands of connections using non-blocking I/O (`epoll`), resulting in lower memory usage and higher concurrency.

**Q: What is a "Reverse Proxy" and why use Nginx for it?**
**A:** A reverse proxy sits in front of backend servers and forwards client requests to them. Nginx is used because it efficiently handles TLS termination (offloading encryption CPU cost from backends), compresses responses (gzip), caches static content, and provides load balancing, allowing backend apps (like Go) to focus purely on business logic.

**Q: How do you perform a zero-downtime reload of Nginx configuration?**
**A:** Run `nginx -s reload`. The master process checks the syntax of the new configuration. If valid, it starts new worker processes with the new config and gracefully shuts down the old workers only after they finish serving their current requests.
