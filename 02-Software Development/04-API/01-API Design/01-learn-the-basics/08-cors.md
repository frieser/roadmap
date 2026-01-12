#API
---
---

## Summary
Cross-Origin Resource Sharing (CORS) is a browser-based security mechanism that allows or restricts web applications from making requests to a domain different from the one that served the web page. It relaxes the **Same-Origin Policy (SOP)** by using specific HTTP headers to permit cross-domain communication.

## Detailed Explanation

### 1. Same-Origin Policy (SOP)
The Same-Origin Policy is a fundamental security model in browsers. It prevents a malicious script on one page from obtaining access to sensitive data on another web page through that page's Document Object Model (DOM). Two URLs have the same origin if the **protocol**, **port**, and **host** are all the same.

### 2. CORS Mechanism
CORS works by adding new HTTP headers that allow servers to describe which origins are permitted to read information from that server.

#### Access-Control-Allow-Origin
This is the most critical header. It tells the browser which domains can access the resource.
- `Access-Control-Allow-Origin: *` (Allows any domain)
- `Access-Control-Allow-Origin: https://example.com` (Allows only example.com)

#### Preflight Requests (OPTIONS)
For certain types of cross-domain requests, the browser first sends an "options" request, called a **preflight request**, to the server. This ensures the server understands the CORS protocol and has given permission for the actual request.
Common headers in preflight:
- `Access-Control-Allow-Methods`: List of allowed HTTP methods.
- `Access-Control-Allow-Headers`: List of allowed custom headers.
- `Access-Control-Max-Age`: How long the preflight response can be cached.

### 3. Simple vs Preflighted Requests

| Feature | Simple Request | Preflighted Request |
| :--- | :--- | :--- |
| **Methods** | GET, HEAD, POST | PUT, DELETE, PATCH, etc. |
| **Headers** | Standard headers (Accept, Language) | Custom headers (X-Auth-Token) |
| **Content-Type** | `text/plain`, `multipart/form-data`, `application/x-www-form-urlencoded` | `application/json`, `application/xml` |
| **Preflight** | No | Yes (OPTIONS request) |

### 4. Handling CORS in Go

#### Using `rs/cors` Middleware
The `rs/cors` package is the most popular library for handling CORS in Go. It integrates easily with `http.Handler`.

```go
package main

import (
	"net/http"
	"github.com/rs/cors"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello, CORS!"))
	})

	// Setup CORS options
	c := cors.New(cors.Options{
		AllowedOrigins:   []string{"https://example.com", "http://localhost:3000"},
		AllowedMethods:   []string{http.MethodGet, http.MethodPost, http.MethodDelete, http.MethodOptions},
		AllowedHeaders:   []string{"Authorization", "Content-Type"},
		AllowCredentials: true,
		Debug:            true, // Useful for development
	})

	// Wrap the mux with CORS middleware
	handler := c.Handler(mux)

	http.ListenAndServe(":8080", handler)
}
```

#### Manual Header Setting
If you don't want to use a library, you can set headers manually in a middleware or handler.

```go
func enableCORS(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Access-Control-Allow-Origin", "https://example.com")
		w.Header().Set("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE")
		w.Header().Set("Access-Control-Allow-Headers", "Accept, Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization")

		// Handle preflight
		if r.Method == http.MethodOptions {
			w.WriteHeader(http.StatusNoContent)
			return
		}

		next.ServeHTTP(w, r)
	})
}
```

## Interview Questions

**Q: What is the main difference between SOP and CORS?**
**A:** SOP (Same-Origin Policy) is a security restriction that blocks cross-origin interactions by default. CORS (Cross-Origin Resource Sharing) is the mechanism that allows us to bypass this restriction safely by providing a set of headers that tell the browser which origins are trusted.

**Q: When exactly is a preflight request triggered?**
**A:** A preflight request is triggered if the request uses an HTTP method other than GET, HEAD, or POST, OR if it uses a Content-Type other than `text/plain`, `multipart/form-data`, or `application/x-www-form-urlencoded`, OR if it contains custom headers.

**Q: Is CORS a server-side security feature?**
**A:** No, CORS is a **browser-side** security feature. The server provides the instructions (headers), but the browser is responsible for enforcing them. If you make a request from a tool like `curl` or a backend server, CORS does not apply.

**Q: What does `Access-Control-Allow-Credentials: true` do?**
**A:** It allows the browser to include credentials (like cookies, authorization headers, or TLS client certificates) in cross-origin requests. If this is true, `Access-Control-Allow-Origin` cannot be set to `*`.
