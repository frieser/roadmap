---
---

## Summary
The **Gang of Four (GoF)** design patterns are 23 classic software design patterns from the book *"Design Patterns: Elements of Reusable Object-Oriented Software"* (1994). They provide standard solutions to common problems in software design, categorized into **Creational** (object creation), **Structural** (class composition), and **Behavioral** (communication between objects).

## Detailed Explanation

While Go is not a traditional Object-Oriented language (no classes or inheritance), many of these patterns are applicable or have idiomatic Go equivalents.

### 1. Creational Patterns
Concerned with the mechanism of object creation.

| Pattern | Description | Go Implementation |
| :--- | :--- | :--- |
| **Singleton** | Ensures a class has only one instance. | `sync.Once` or package-level variables. |
| **Factory Method** | Defines an interface for creating objects but lets subclasses decide which class to instantiate. | Functions returning interfaces (e.g., `NewClient() Client`). |
| **Abstract Factory** | Creates families of related objects without specifying concrete classes. | Interfaces returning other interfaces. |
| **Builder** | Separates object construction from its representation. | Chained methods or "Functional Options" pattern (idiomatic Go). |
| **Prototype** | Creates new objects by copying an existing one. | Struct copying (`val := *original`) or deep copy methods. |

### 2. Structural Patterns
Concerned with how classes and objects are composed.

| Pattern | Description | Go Implementation |
| :--- | :--- | :--- |
| **Adapter** | Converts one interface to another. | Wrapper struct implementing the target interface. |
| **Bridge** | Decouples an abstraction from its implementation. | Interface fields within structs. |
| **Composite** | Tree structure of objects (part-whole hierarchy). | Struct containing a slice of its own interface type. |
| **Decorator** | Adds behavior dynamically. | Embedding interfaces or wrapping interfaces (Middleware). |
| **Facade** | Simple interface to a complex subsystem. | A high-level package or struct API. |
| **Flyweight** | Shares common state to support large numbers of objects. | Pointers to shared immutable structs. |
| **Proxy** | Placeholder to control access to an object. | Wrapper struct (e.g., for lazy loading or caching). |

### 3. Behavioral Patterns
Concerned with algorithms and assignment of responsibilities.

| Pattern | Description | Go Implementation |
| :--- | :--- | :--- |
| **Chain of Responsibility** | Passes requests along a chain of handlers. | Middleware chains (e.g., `http.Handler` chaining). |
| **Command** | Encapsulates a request as an object. | Function types or interfaces with `Execute()` method. |
| **Interpreter** | Evaluates sentences in a language. | Abstract Syntax Tree (AST) parsers (`go/ast`). |
| **Iterator** | Traverses a collection. | `for range` loops or `Next()` methods. |
| **Mediator** | Centralizes communication between objects. | Channels (`chan`) are built-in mediators. |
| **Memento** | Captures and restores internal state. | Serializing/saving state structs. |
| **Observer** | Notifies dependents of state changes. | Channels or callback functions. |
| **State** | Alters behavior when internal state changes. | Interfaces representing state behaviors. |
| **Strategy** | Encapsulates interchangeable algorithms. | First-class functions or interfaces. |
| **Template Method** | Defines skeleton of algorithm. | Not idiomatic (requires inheritance); usually replaced by Composition/Strategy. |
| **Visitor** | Separates algorithm from object structure. | Interfaces, but rigid; less common in Go. |

## Go Example: Functional Options (Builder Variant)

One of the most famous Go idioms replacing the complex Builder pattern.

```go
package main

import "fmt"

type Server struct {
	Host string
	Port int
	TLS  bool
}

// Option function type
type Option func(*Server)

func NewServer(opts ...Option) *Server {
	// Default values
	s := &Server{
		Host: "localhost",
		Port: 8080,
		TLS:  false,
	}
	
	// Apply options
	for _, opt := range opts {
		opt(s)
	}
	return s
}

func WithPort(port int) Option {
	return func(s *Server) {
		s.Port = port
	}
}

func WithTLS() Option {
	return func(s *Server) {
		s.TLS = true
	}
}

func main() {
	// Clean, readable, flexible construction
	srv := NewServer(
		WithPort(9000),
		WithTLS(),
	)
	fmt.Printf("Server running on %s:%d (TLS: %v)\n", srv.Host, srv.Port, srv.TLS)
}
```

## Interview Questions

### Q: Why is the Singleton pattern often discouraged in Go?
**A:** Singletons introduce global state, making code harder to test (difficult to mock) and harder to reason about in concurrent environments. Dependency Injection is preferred. If needed, `sync.Once` is the thread-safe way to implement it.

### Q: How does Go implement the Decorator pattern?
**A:** Go uses **interface wrapping**. For example, `http.Handler` middleware wraps an existing handler, adds functionality (logging, auth), and calls the underlying handler.

### Q: Which GoF patterns are built into the Go language?
**A:**
*   **Observer**: Channels (`chan`).
*   **Iterator**: `range` keyword.
*   **Proxy/Decorator**: Embedding and Interfaces.
*   **Strategy**: Functions as first-class citizens.
