---
---

## Summary
**Dependency Injection (DI)** is a technique where an object receives other objects that it depends on (dependencies) from the outside, rather than creating them internally. This implements the **Inversion of Control (IoC)** principle. In Go, DI is critical for writing testable, loosely coupled code.

## Detailed Explanation

### The Problem (Tight Coupling)
Without DI, a function creates its own dependencies.
```go
func CreateUser() {
    db := sql.Open(...) // Hardcoded dependency!
    db.Exec(...)
}
```
This is hard to test (requires a real DB) and hard to change.

### The Solution (Dependency Injection)
With DI, the dependency is passed in (usually via Constructor).
```go
func CreateUser(db DBInterface) {
    db.Exec(...)
}
```
Now we can pass a real DB in production and a **Mock DB** in tests.

### Types of DI
1.  **Constructor Injection** (Idiomatic Go): Dependencies are passed to the "constructor" function (e.g., `NewService(...)`).
2.  **Setter Injection**: Dependencies are set via methods after creation. Less common in Go as it allows partially initialized objects.
3.  **Interface Injection**: The dependency provides an injector method that will inject the dependency into any client that passes itself (rare).

### DI Containers
While **Manual Wiring** (writing the dependency graph in `main.go`) is preferred for simple to medium Go apps, larger projects may use tools:
*   **Google Wire**: A code-generation tool. It verifies the graph at **compile time**. (Preferred).
*   **Uber Fx**: A reflection-based framework. It resolves dependencies at **runtime**. (Magical, harder to debug).
*   **Dig**: Also reflection-based (underlies Fx).

## Go Example: Constructor Injection

```go
package main

import "fmt"

// 1. Define the dependency as an Interface
type Logger interface {
	Log(msg string)
}

// 2. Concrete implementation
type ConsoleLogger struct{}

func (c *ConsoleLogger) Log(msg string) {
	fmt.Println("[Log]:", msg)
}

// 3. The Client (Service) depends on the Interface
type OrderService struct {
	logger Logger // Dependency
}

// 4. Constructor Injection
// We force the caller to provide the dependency
func NewOrderService(l Logger) *OrderService {
	return &OrderService{
		logger: l,
	}
}

func (s *OrderService) PlaceOrder(id string) {
	s.logger.Log("Order placed: " + id)
}

// 5. Wiring in Main
func main() {
	// Create dependency
	logger := &ConsoleLogger{}

	// Inject dependency
	service := NewOrderService(logger)

	service.PlaceOrder("ORD-123")
}
```

## Interview Questions

### Q: Why is Constructor Injection preferred over Setter Injection in Go?
**A:** Constructor Injection guarantees that an object is **fully initialized** and ready to use upon creation. It enforces required dependencies at compile time (you can't call `NewService` without arguments). Setter injection leaves objects in a potentially invalid state (nil pointers) until configured.

### Q: What is the difference between Manual Wiring and using a DI Container?
**A:**
*   **Manual Wiring**: You explicitly write `NewServiceA(NewRepoB(db))` in your `main.go`. It's explicit, readable, and compiler-checked.
*   **DI Container**: The container automatically discovers and connects components. It reduces boilerplate code but introduces "magic" and can make it harder to trace which implementation is actually being used.

### Q: How does DI enable Unit Testing?
**A:** By depending on interfaces rather than concrete types, DI allows tests to inject **Mock** implementations. A mock can simulate specific scenarios (e.g., database error, network timeout) without needing the real infrastructure, making tests fast and deterministic.
