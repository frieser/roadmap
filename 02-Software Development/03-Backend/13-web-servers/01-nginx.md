---
---

# Nginx

## 1. Summary
Nginx is a high-performance, open-source web server that also functions as a reverse proxy, load balancer, and HTTP cache. It is designed for maximum performance and stability, using an asynchronous, event-driven architecture that allows it to handle thousands of concurrent connections with minimal memory footprint.

## 2. Detailed Explanation
Nginx (pronounced "engine-x") was created to solve the "C10k problem" (handling 10,000 concurrent connections).

### Architecture: Event-driven vs. Thread-based
- **Event-driven (Nginx)**: Uses a single-threaded event loop per worker process. It handles thousands of connections asynchronously using non-blocking I/O (e.g., `epoll` on Linux). It does not block while waiting for data, allowing it to move to the next request immediately.
- **Thread-based (Traditional)**: Spawns a new thread or process for every connection. This leads to high memory overhead (stack per thread) and CPU context switching when scaling (C10k bottleneck).

### Master-Worker Model:
- **Master Process**: Handles privileged operations (reading config, binding ports) and manages worker processes.
- **Worker Processes**: Single-threaded loops that handle connections using non-blocking I/O.

## 3. Go-specific Context & Examples

### Using Nginx Unit with Go
Nginx Unit is a polyglot application server. To run Go apps natively on Unit, use the `unit.nginx.org/go` package.

```go
package main

import (
    "io"
    "net/http"
    "unit.nginx.org/go" // Native Unit support
)

func handler(w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, "Hello from Go on Nginx Unit!")
}

func main() {
    http.HandleFunc("/", handler)
    // Unit handles the port binding via JSON configuration
    unit.ListenAndServe(":8080", nil) 
}
```

### Writing Custom Modules in Go (Conceptual)
Traditional Nginx modules are written in C. For Go, the recommended modern approach is **Proxy-Wasm**.
- **Proxy-Wasm**: Write Go code that compiles to WebAssembly (WASM). This can be run inside Nginx using the `wasm-nginx-module`. This avoids the complexities of CGO and Nginx's memory management.

### Reverse Proxy Configuration
```nginx
server {
    listen 80;
    server_name my-go-app.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 4. Interview Questions
1. **What is the difference between Nginx and Apache's architecture?**
   - Nginx is asynchronous and event-driven; Apache traditionally uses a thread-per-connection model (though it now supports Event MPM).
2. **How does Nginx handle the C10k problem?**
   - By using a non-blocking event loop instead of spawning a thread per connection.
3. **What is Nginx Unit?**
   - A dynamic application server that supports multiple languages (Go, Python, PHP) and uses a JSON API for configuration.
