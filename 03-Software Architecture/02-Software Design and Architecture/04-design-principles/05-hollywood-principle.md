---
---

## Summary
The **Hollywood Principle** states: **"Don't call us, we'll call you."** It is the defining characteristic of a **Framework** vs. a **Library**. In architecture, it relates to **Inversion of Control (IoC)**, where the overall flow of control is dictated by the framework, which calls into your custom code (plugins/handlers) at specific extension points.

## Detailed Explanation

### 1. Library vs. Framework
*   **Library**: You call the library. You are in control. (e.g., `math.Sqrt()`, `http.Get()`).
*   **Framework**: The framework calls you. It controls the lifecycle. (e.g., `http.ListenAndServe` calls your `ServeHTTP` method).

### 2. Inversion of Control (IoC)
The Hollywood Principle is essentially IoC. Instead of the high-level policy controlling the low-level details directly, the control is inverted. The high-level policy defines an interface (hook), and the low-level detail is "called" when needed.

### 3. Benefits
*   **Decoupling**: The framework doesn't need to know about your specific implementation, only your interface.
*   **Extensibility**: You can extend the framework's behavior without modifying its source code.

## Go Application (Interfaces & Callbacks)

### HTTP Handlers (The Classic Example)
The standard `net/http` package in Go uses the Hollywood Principle. You don't write the loop that accepts TCP connections. You just write the handler, and the server calls you.

```go
// "Don't call us" (The Server Logic)
// The HTTP Server is pre-written. It knows how to accept connections.

// "We'll call you" (Your Handler)
type MyHandler struct{}

func (h *MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello via Hollywood Principle!")
}

func main() {
    handler := &MyHandler{}
    // We register the handler, giving control to the server
    http.ListenAndServe(":8080", handler)
}
```

### Dependency Injection Containers (Uber fx, Google Wire)
These frameworks instantiate your objects for you. You don't call `NewService()`; the container calls your constructor when it determines a dependency is needed.

## Interview Questions

**Q: How does the Template Method design pattern relate to the Hollywood Principle?**
**A:** The Template Method defines the skeleton of an algorithm in a base class (or function) but lets subclasses (or callbacks) override specific steps. The base algorithm controls the flow and "calls" the custom steps. This is a direct application of the Hollywood Principle.

**Q: What is the downside of this principle?**
**A:** "IoC Hell" or "Callback Hell". The flow of control becomes implicit. You can't just read the `main()` function to see what happens sequentially. You have to understand the framework's lifecycle to know *when* your code will be executed, which makes debugging harder.
