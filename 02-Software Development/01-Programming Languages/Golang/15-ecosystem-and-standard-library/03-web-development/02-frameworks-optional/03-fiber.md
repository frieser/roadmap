# Fiber Web Framework

## Summary
Fiber is an Express.js-inspired web framework built on top of **Fasthttp**, the fastest HTTP engine for Go. It prioritizes **Zero Allocation** and extreme performance. Because it uses Fasthttp instead of `net/http`, it is not directly compatible with standard library middleware, but it offers blistering speed for high-throughput applications.

## Detailed Explanation

### 1. Why Fiber?
*   **Express.js Style**: If you come from Node.js, Fiber feels very familiar (`app.Get`, `app.Post`).
*   **Performance**: It is consistently one of the fastest Go frameworks in benchmarks due to aggressive memory optimization (e.g., reusing request contexts).

### 2. Basic Usage
```go
package main

import "github.com/gofiber/fiber/v2"

func main() {
    app := fiber.New()

    app.Get("/", func(c *fiber.Ctx) error {
        return c.SendString("Hello, World!")
    })

    app.Listen(":3000")
}
```

### 3. Zero Allocation
Fiber minimizes memory allocation by returning byte slices that reference the underlying buffer.
**Warning**: Because buffers are reused, the values in `c.Body()` or `c.Params()` are only valid *during* the handler execution. If you need to keep data after the handler returns (e.g., in a goroutine), you must copy it using `utils.CopyString()`.

## Interview Questions

**Q: What is the major trade-off when choosing Fiber over Gin or Echo?**
**A:** Compatibility. Fiber is built on `fasthttp`, not `net/http`. This means you cannot use the vast ecosystem of standard Golang middleware (like standard authentication handlers or `gorilla/websocket`) directly without wrappers. You are locked into the Fiber/Fasthttp ecosystem.

**Q: Why is Fiber faster than `net/http`?**
**A:** `net/http` allocates a new goroutine and several objects for every request. `fasthttp` (and thus Fiber) uses worker pools and reuses request/response objects to reduce Garbage Collection (GC) pressure. This "zero allocation" strategy yields higher throughput but requires more careful coding to avoid race conditions with reused buffers.

**Q: How does Fiber handle "unsafe" memory usage?**
**A:** By default, Fiber (via Fasthttp) returns strings/bytes that point to a reusable buffer. If you launch a goroutine inside a handler and access `c.Params("id")`, the value might change as the buffer is reused for the next request. You must make an immutable copy of the data before passing it to asynchronous processes.
