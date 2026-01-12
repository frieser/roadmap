---
---

## Summary
The **Gang of Four (GoF)** patterns are the 23 foundational design patterns published in the book *"Design Patterns: Elements of Reusable Object-Oriented Software"* (1994). For a Software Architect, these are the vocabulary of communication. They are categorized into **Creational** (object creation), **Structural** (class composition), and **Behavioral** (object communication).

## Detailed Explanation

### 1. Creational Patterns
Concerned with the process of object creation. They decouple a system from how its objects are created, composed, and represented.

*   **Factory Method**: Defines an interface for creating an object, but lets subclasses decide which class to instantiate. (Dependency Injection is the modern evolution of this).
*   **Singleton**: Ensures a class has only one instance. *Warning*: Often an anti-pattern (Global State). Use DI instead.
*   **Builder**: Separates the construction of a complex object from its representation. (Essential for complex structs in Go).

### 2. Structural Patterns
Concerned with how classes and objects are composed to form larger structures.

*   **Adapter**: Converts the interface of a class into another interface clients expect. (Crucial for Hexagonal Architecture).
*   **Facade**: Provides a unified interface to a set of interfaces in a subsystem. (The entry point to a Module/Service).
*   **Decorator**: Attaches additional responsibilities to an object dynamically. (Middleware in web servers).
*   **Proxy**: Provides a surrogate or placeholder for another object to control access to it. (Lazy loading, Security).

### 3. Behavioral Patterns
Concerned with algorithms and the assignment of responsibilities between objects.

*   **Strategy**: Defines a family of algorithms, encapsulates each one, and makes them interchangeable. (The "Open/Closed" enabler).
*   **Observer**: One-to-many dependency so that when one object changes state, all its dependents are notified. (Event-Driven Architecture).
*   **Command**: Encapsulates a request as an object. (Job queues, Undo operations).
*   **Template Method**: Defines the skeleton of an algorithm in an operation, deferring some steps to subclasses.

## Go Code Examples

### Strategy Pattern (Behavioral)
Swapping logic at runtime without changing code.

```go
type PaymentStrategy interface {
    Pay(amount float64) error
}

type CreditCard struct{}
func (c *CreditCard) Pay(amount float64) error { return nil }

type PayPal struct{}
func (p *PayPal) Pay(amount float64) error { return nil }

// Context
type ShoppingCart struct {
    Payer PaymentStrategy // Injected
}
```

### Decorator Pattern (Structural)
Standard Go middleware pattern.

```go
type Handler func(string)

func LogMiddleware(next Handler) Handler {
    return func(msg string) {
        fmt.Println("Log: Before")
        next(msg)
        fmt.Println("Log: After")
    }
}

func Hello(msg string) { fmt.Println(msg) }

// Usage
decorated := LogMiddleware(Hello)
decorated("World")
```

### Functional Options (Builder Variation)
The idiomatic Go replacement for the Builder pattern.

```go
type Server struct {
    Port int
    Timeout int
}

type Option func(*Server)

func WithPort(p int) Option {
    return func(s *Server) { s.Port = p }
}

func NewServer(opts ...Option) *Server {
    s := &Server{Port: 8080, Timeout: 30} // Defaults
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

## Interview Questions

**Q: Which GoF pattern is most essential for unit testing?**
**A:** **Dependency Injection** (an evolution of Factory/Strategy). By accepting interfaces (Strategies) rather than creating concrete objects, you can swap real implementations for Mocks/Stubs during testing.

**Q: Is Singleton considered an Anti-Pattern?**
**A:** Yes, in modern architecture. Singletons introduce Global State, which makes testing difficult (state persists between tests) and hides dependencies (you don't know a function needs the DB just by looking at its signature). Dependency Injection Singletons (lifetime managed by the DI container) are preferred over static Singletons.

**Q: Difference between Adapter and Facade?**
**A:** 
*   **Adapter**: Makes two *incompatible* interfaces work together. It wraps an existing object to match a required interface.
*   **Facade**: Simplifies a *complex* subsystem behind a new, simpler interface. It doesn't fix incompatibility; it fixes complexity.
