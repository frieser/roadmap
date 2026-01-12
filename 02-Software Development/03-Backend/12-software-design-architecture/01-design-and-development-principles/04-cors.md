---
---

## Summary
Cross-Origin Resource Sharing (CORS) is a security mechanism that allows a web page from one domain to request resources (like APIs, fonts, or images) from another domain. It relaxes the browser's Same-Origin Policy (SOP), which by default restricts scripts from making cross-origin HTTP requests. CORS uses HTTP headers to tell the browser whether the server permits the request.

## Detailed Explanation

### The Same-Origin Policy (SOP)
By default, browsers enforce the SOP, meaning a script running on `domain-a.com` cannot make an AJAX/Fetch request to `domain-b.com`. This prevents malicious sites from acting on behalf of a user to a different site where they are logged in.

### How CORS Works
CORS works by adding specific HTTP headers that allow the server to define which origins are permitted to access its resources.

1.  **Simple Requests**: For GET, HEAD, or POST requests with standard headers, the browser sends the request with an `Origin` header. The server responds with `Access-Control-Allow-Origin`. If the origin matches, the browser allows the response.
2.  **Preflight Requests**: For "complex" requests (e.g., using PUT/DELETE, or custom headers like `Authorization`), the browser first sends an `OPTIONS` request to verify safety.
    *   **Browser sends**: `Origin`, `Access-Control-Request-Method`, `Access-Control-Request-Headers`.
    *   **Server responds**: `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, `Access-Control-Max-Age`.

### Key Headers
*   `Access-Control-Allow-Origin`: Specifies which origin(s) can access the resource (e.g., `*` or `https://example.com`).
*   `Access-Control-Allow-Methods`: Specifies allowed HTTP methods (e.g., `GET, POST, OPTIONS`).
*   `Access-Control-Allow-Headers`: Specifies allowed custom headers.
*   `Access-Control-Allow-Credentials`: Indicates whether the browser should send cookies/auth headers.

## Go-Specific Context/Examples

In Go, you can handle CORS manually by setting headers in your handler, or more commonly, by using a middleware library like `rs/cors` or writing your own middleware.

### Example: Custom CORS Middleware
```go
package main

import (
	"fmt"
	"net/http"
)

func corsMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Set CORS headers
		w.Header().Set("Access-Control-Allow-Origin", "*") // Allow all origins (use specific domain in prod)
		w.Header().Set("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE")
		w.Header().Set("Access-Control-Allow-Headers", "Accept, Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization")

		// Handle Preflight OPTIONS request
		if r.Method == "OPTIONS" {
			w.WriteHeader(http.StatusOK)
			return
		}

		next.ServeHTTP(w, r)
	})
}

func mainHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, CORS is enabled!")
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", mainHandler)

	fmt.Println("Server starting on :8080")
	http.ListenAndServe(":8080", corsMiddleware(mux))
}
```

### Using `rs/cors` Library (Standard)
This is the most popular library for handling CORS in Go, offering granular control.

```go
package main

import (
	"net/http"
	"github.com/rs/cors"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello!"))
	})

	// Configure CORS handler
	c := cors.New(cors.Options{
		AllowedOrigins:   []string{"http://localhost:3000", "https://mydomain.com"},
		AllowedMethods:   []string{"GET", "POST", "PUT", "DELETE"},
		AllowedHeaders:   []string{"Authorization", "Content-Type"},
		AllowCredentials: true,
		Debug:            true,
	})

	handler := c.Handler(mux)
	http.ListenAndServe(":8080", handler)
}
```

## Interview Questions

**Q: What is the difference between a simple request and a preflight request in CORS?**
**A:** A simple request (HEAD, GET, POST with standard headers) is sent directly. A preflight request is an `OPTIONS` request sent *before* the actual request when using non-standard methods (PUT, DELETE) or headers, asking the server for permission to proceed.

**Q: What is the security risk of setting `Access-Control-Allow-Origin: *`?**
**A:** It allows any website to request resources from your server via the user's browser. While fine for public APIs, it is dangerous for internal APIs relying on cookies or auth tokens, as it bypasses the Same-Origin Policy protections against some attacks (though CSRF tokens are still needed). Note that `*` cannot be used if `Allow-Credentials` is true.

**Q: How do you handle CORS when your frontend and backend are on different subdomains (e.g., `app.site.com` and `api.site.com`)?**
**A:** You must explicitly allow the frontend origin (`https://app.site.com`) in the backend's `Access-Control-Allow-Origin` header. You cannot rely on default SOP, and if you need to share cookies (session), you must also set `Access-Control-Allow-Credentials: true`.
