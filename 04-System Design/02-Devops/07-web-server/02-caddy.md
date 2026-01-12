---
---

# Caddy

Caddy is a modern, open-source web server written in **Go**. It is famous for being the only web server that uses **HTTPS by default** (automatically obtaining and renewing certificates via Let's Encrypt). Its simplicity and "batteries-included" philosophy make it a favorite for modern DevOps workflows.

## Summary

Caddy is designed for ease of use. It automatically manages TLS certificates, supports HTTP/3 (QUIC) out of the box, and is configured via a simple `Caddyfile` or a powerful JSON API. Because it is written in Go, it is a single static binary with no external dependencies, making it perfect for containerized environments.

## Detailed Explanation

### 1. Key Features
*   **Automatic HTTPS**: Automatically provisions certificates from Let's Encrypt or ZeroSSL and renews them. No cron jobs or certbot required.
*   **Memory Safe**: Written in Go, protecting against buffer overflows common in C-based servers.
*   **Extensible**: Modular architecture allows you to compile plugins directly into the binary (e.g., specialized DNS providers, auth modules).
*   **Configuration**:
    *   **Caddyfile**: Human-readable, simple syntax.
    *   **JSON API**: For dynamic, programmatic configuration updates without restarts.

### 2. Architecture
Like Nginx, Caddy is event-driven (Go routines). It uses the Go runtime's scheduler to handle high concurrency efficiently across CPU cores.

---

## Go Implementation Example

Since Caddy is written in Go, you can extend it by writing **Modules**. A module allows you to add custom logic (e.g., a custom authentication middleware) directly into the web server.

### Creating a Custom Caddy Module
This example creates a simple "Visitor Count" middleware.

```go
package visitor_count

import (
	"fmt"
	"net/http"
	"sync"

	"github.com/caddyserver/caddy/v2"
	"github.com/caddyserver/caddy/v2/caddyconfig/caddyfile"
	"github.com/caddyserver/caddy/v2/caddyconfig/httpcaddyfile"
	"github.com/caddyserver/caddy/v2/modules/caddyhttp"
)

func init() {
	caddy.RegisterModule(VisitorCount{})
	httpcaddyfile.RegisterHandlerDirective("visitor_count", parseCaddyfile)
}

// VisitorCount implements caddyhttp.MiddlewareHandler
type VisitorCount struct {
	count int64
	mu    sync.Mutex
}

// CaddyModule returns the Caddy module information.
func (VisitorCount) CaddyModule() caddy.ModuleInfo {
	return caddy.ModuleInfo{
		ID:  "http.handlers.visitor_count",
		New: func() caddy.Module interface{} { return new(VisitorCount) },
	}
}

// ServeHTTP implements the middleware logic.
func (v *VisitorCount) ServeHTTP(w http.ResponseWriter, r *http.Request, next caddyhttp.Handler) error {
	v.mu.Lock()
	v.count++
	current := v.count
	v.mu.Unlock()

	// Add header to response
	w.Header().Set("X-Visitor-Count", fmt.Sprintf("%d", current))

	return next.ServeHTTP(w, r)
}

// Provision sets up the module (optional)
func (v *VisitorCount) Provision(ctx caddy.Context) error {
	return nil
}

// UnmarshalCaddyfile sets up the handler from Caddyfile tokens
func parseCaddyfile(h httpcaddyfile.Helper) (caddyhttp.MiddlewareHandler, error) {
	return new(VisitorCount), nil
}
```

## Interview Questions

**Q: Why would you choose Caddy over Nginx?**
**A:** Choose Caddy for:
1.  **Automatic HTTPS**: If you don't want to manage `certbot` or certificate renewal scripts.
2.  **Simplicity**: The `Caddyfile` is significantly easier to read and write than Nginx config.
3.  **Memory Safety**: If security (memory corruption vulnerabilities) is a primary concern.
4.  **Go Integration**: If your team is already comfortable with Go and wants to write custom plugins in the same language.

**Q: How does Caddy handle "Zero Downtime" configuration changes?**
**A:** Caddy exposes an administrative JSON API (localhost:2019). You can POST a new JSON configuration to the `/load` endpoint. Caddy will start the new configuration in the background, transfer the listening sockets from the old config to the new one, and then gracefully shut down the old config—all without dropping a single connection.

**Q: What is the significance of HTTP/3 support in Caddy?**
**A:** Caddy was one of the first servers to support HTTP/3 (QUIC) by default. HTTP/3 runs over UDP (instead of TCP) and solves the "Head-of-Line Blocking" problem, significantly improving performance on unreliable networks (like mobile data).
