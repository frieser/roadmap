---
---

# Caddy

## 1. Summary
Caddy is a modern, open-source web server written in **Go**. It is famous for being the first web server to provide **Automatic HTTPS** by default using Let's Encrypt or ZeroSSL. It is highly extensible via plugins and features a simple, declarative configuration format known as the Caddyfile.

## 2. Detailed Explanation
Caddy is built on top of Go's standard library (`net/http`) but extends it with enterprise features like automatic TLS, HTTP/3 support, and a JSON-first configuration engine.

### Key Features:
- **Automatic HTTPS**: Manages certificate issuance, renewal, and OCSP stapling automatically.
- **Caddyfile**: A human-readable configuration format that is often much shorter than Nginx or Apache configs.
- **On-Demand TLS**: Can issue certificates for domains as they are requested (useful for SaaS platforms).

## 3. Go-specific Context & Examples

### Extending Caddy with a Custom Module
Since Caddy is written in Go, you can extend it by implementing the `caddy.Module` interface.

```go
package visitorcounter

import (
	"fmt"
	"net/http"
	"sync/atomic"

	"github.com/caddyserver/caddy/v2"
	"github.com/caddyserver/caddy/v2/modules/caddyhttp"
)

func init() {
	caddy.RegisterModule(VisitorCounter{})
}

type VisitorCounter struct {
	HeaderName string `json:"header_name,omitempty"`
	count      uint64
}

func (VisitorCounter) CaddyModule() caddy.ModuleInfo {
	return caddy.ModuleInfo{
		ID:  "http.handlers.visitor_counter",
		New: func() caddy.Module { return new(VisitorCounter) },
	}
}

func (v VisitorCounter) ServeHTTP(w http.ResponseWriter, r *http.Request, next caddyhttp.Handler) error {
	current := atomic.AddUint64(&v.count, 1)
	w.Header().Set(v.HeaderName, fmt.Sprintf("%d", current))
	return next.ServeHTTP(w, r)
}
```

### Building with `xcaddy`
To include your module, build Caddy using the `xcaddy` tool:
```bash
xcaddy build --with github.com/user/visitor_counter
```

## 4. Interview Questions
1. **What language is Caddy written in, and why does it matter?**
   - Written in Go. This allows Go developers to extend it easily and ensures "single binary" deployment.
2. **How does Caddy handle SSL certificates?**
   - Automatically via Let's Encrypt or ZeroSSL, including renewal and OCSP stapling.
3. **What is the Caddyfile?**
   - An easy-to-read configuration format that maps to Caddy's underlying JSON configuration.
