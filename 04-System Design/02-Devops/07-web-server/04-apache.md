---
---

# Apache HTTP Server (httpd)

The Apache HTTP Server is one of the oldest and most widely used web servers in history. Known for its modularity and flexibility, it remains a staple in enterprise environments and legacy applications, particularly those relying on the **LAMP** stack (Linux, Apache, MySQL, PHP).

## Summary

Apache uses a modular architecture where functionality (like SSL, Rewrite rules, PHP execution) is added via dynamically loaded modules. Its defining feature is the `.htaccess` file, which allows decentralized configuration at the directory level—a feature Nginx explicitly avoids for performance reasons.

## Detailed Explanation

### 1. Architecture: MPMs (Multi-Processing Modules)
Apache's performance depends on the chosen MPM:
*   **Prefork**: Processes-based. Each request gets a dedicated process. Safe for non-thread-safe libraries (like old PHP) but memory-hungry.
*   **Worker**: Hybrid. Uses multiple processes, each with multiple threads. Handles concurrency better than Prefork.
*   **Event**: The modern standard. Similar to Nginx, it uses a dedicated thread to handle keep-alive connections and passes requests to worker threads only when necessary.

### 2. .htaccess
A configuration file placed inside the web root.
*   **Pros**: Allows developers to change rules (redirects, password protection) without root access to the main server config.
*   **Cons**: Performance penalty. The server must check for `.htaccess` in every directory up the path for every request.

---

## Go Implementation Example

Unlike Nginx or Caddy, you typically don't extend Apache with Go. Instead, you use Apache as a **Reverse Proxy** in front of a Go application. This setup allows Apache to handle SSL, static files, and legacy PHP apps while forwarding API traffic to Go.

### Apache Configuration for Go Reverse Proxy
*(Not Go code, but the configuration required to run Go behind Apache)*

```apache
<VirtualHost *:443>
    ServerName api.example.com
    
    # Enable SSL
    SSLEngine on
    SSLCertificateFile /path/to/cert.pem
    SSLCertificateKeyFile /path/to/key.pem

    # Preserve Host header so Go knows the real domain
    ProxyPreserveHost On

    # Forward all requests to the Go app running on localhost:8080
    ProxyPass / http://localhost:8080/
    ProxyPassReverse / http://localhost:8080/
</VirtualHost>
```

### Go Application
The Go app acts as a simple backend service.

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	// Simple backend that trusts Apache to handle SSL
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// Apache sends the real client IP in X-Forwarded-For
		clientIP := r.Header.Get("X-Forwarded-For")
		fmt.Fprintf(w, "Hello from Go! Your IP (via Apache): %s", clientIP)
	})

	// Listen on localhost only, so only Apache can talk to us
	http.ListenAndServe("127.0.0.1:8080", nil)
}
```

## Interview Questions

**Q: When would you choose Apache over Nginx?**
**A:**
1.  **Shared Hosting**: If you need to let users configure their own directories using `.htaccess` without restarting the server.
2.  **Legacy Modules**: If you rely on specific Apache modules (e.g., `mod_security`, `mod_perl`) that don't have Nginx equivalents.
3.  **Dynamic Content**: Apache can process dynamic content (PHP via `mod_php`) internally, whereas Nginx must pass it to an external processor (PHP-FPM).

**Q: Explain the performance impact of `.htaccess`.**
**A:** When `AllowOverride` is enabled (permitting `.htaccess`), Apache looks for this file in every directory of the requested path. For a request to `/var/www/site/images/logo.png`, it checks `/`, `/var`, `/var/www`, etc. This adds significant disk I/O latency. For high performance, `.htaccess` should be disabled (`AllowOverride None`).

**Q: What is the difference between `MPM Prefork` and `MPM Event`?**
**A:** `Prefork` creates a separate process for each connection. It is stable but memory-intensive and doesn't scale well with keep-alive connections. `Event` uses threads and a dedicated listener thread to manage keep-alive connections, handing them off to worker threads only when a request actually arrives. This mimics Nginx's efficiency while keeping Apache's compatibility.
