# net/http Standard Library

## Summary
Go's `net/http` package is a production-grade HTTP client and server implementation. Unlike many other languages where you need a third-party server (like Apache/Nginx or WSGI), Go applications serve HTTP traffic directly. It is robust, supports HTTP/2 out of the box, and uses a simple, blocking I/O model (goroutine per request) that scales effortlessly to thousands of concurrent connections.

## Detailed Explanation

### 1. The Core Interface: `http.Handler`
Everything in Go web development revolves around this interface:
```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```
Any object that implements `ServeHTTP` can handle a request.

### 2. Starting a Server
```go
package main

import (
    "fmt"
    "net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

func main() {
    // 1. Register handler to a path
    http.HandleFunc("/", helloHandler)

    // 2. Start server (blocks forever)
    // nil means use the DefaultServeMux
    http.ListenAndServe(":8080", nil)
}
```

### 3. `http.HandlerFunc` (The Adapter)
You typically write functions, not structs. `HandlerFunc` adapts a function to satisfy the `Handler` interface.
```go
// The type
type HandlerFunc func(ResponseWriter, *Request)

// The method that makes it a Handler
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

### 4. Middleware Pattern
Middleware is just a function that takes a `Handler` and returns a `Handler`, wrapping logic around it.
```go
func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("Request received:", r.URL.Path)
        next.ServeHTTP(w, r) // Call the next handler
    })
}
```

### 5. The HTTP Client
```go
resp, err := http.Get("http://example.com/")
if err != nil {
    // handle error
}
defer resp.Body.Close() // CRITICAL: Must close body to prevent leaks

body, _ := io.ReadAll(resp.Body)
```

## Interview Questions

**Q: What is `http.DefaultServeMux`?**
**A:** It is a global `ServeMux` (router) instance stored in the `net/http` package. When you call `http.HandleFunc("/path", handler)`, it registers the handler to this global multiplexer. While convenient for simple scripts, it is often avoided in production libraries to prevent route collisions with other packages.

**Q: Why is it important to close `resp.Body` when using the HTTP client?**
**A:** The standard library uses persistent TCP connections (Keep-Alive) by default. If you don't close the response body (and read it until EOF), the underlying TCP connection cannot be reused for subsequent requests, leading to resource leaks (file descriptors) and exhaustion of available ports.

**Q: How does `net/http` handle concurrent requests?**
**A:** It spawns a new goroutine for every incoming request. This allows the server to handle thousands of requests concurrently without blocking, utilizing Go's efficient scheduler. However, this also means your handlers must be thread-safe (e.g., use Mutexes when accessing shared global state).
