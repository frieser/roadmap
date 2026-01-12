---
---

## Summary
Application Architecture focuses on the internal design and organization of a single software application or service. Its goal is to create code that is modular, testable, and maintainable by defining clear boundaries between business logic, data access, and external interfaces.

## Detailed Explanation

Application Architecture deals with the "Deep" view of a single repository. It answers questions like: "How do I organize my packages?", "How do I handle database transactions?", and "Where does the validation logic go?".

### Key Concepts

1.  **Layering**: Dividing the application into logical layers (Presentation, Domain, Data) to separate concerns.
2.  **Design Patterns**: applying standard solutions to common coding problems (e.g., Singleton, Factory, Strategy).
3.  **State Management**: How the application handles data in memory and persistence.
4.  **Modularity**: Ensuring high cohesion (related things stay together) and low coupling (independent things stay apart).

### Common Styles
-   **Layered Architecture**: Traditional N-tier (Controller -> Service -> Repository).
-   **Hexagonal (Ports & Adapters)**: Isolates the core domain from technical details using interfaces.
-   **Clean Architecture**: A variant of Hexagonal, emphasizing strict dependency rules (inwards only).

### Application in Go (Golang)

Go is particularly suited for **Hexagonal Architecture** because its implicit interface implementation makes it easy to decouple layers.

#### 1. The "Internal" Package
Go provides a language-level mechanism for encapsulation: the `internal/` directory. Packages inside `internal/` cannot be imported by code outside the parent directory. This enforces architectural boundaries by preventing external services from importing your private logic.

#### 2. Dependency Injection (DI)
In Go, DI is usually done explicitly via constructor injection, often aided by libraries like `uber-fx` or `google/wire` for complex apps, or just manual composition for simpler ones.

```go
// Application Architecture Example: Hexagonal Style in Go

// DOMAIN (Core logic - No dependencies)
type Order struct {
    ID     string
    Amount float64
}

// PORT (Interface defining what the domain needs)
type OrderRepository interface {
    Save(ctx context.Context, order Order) error
}

// SERVICE (Application Logic - Depends on Port)
type OrderService struct {
    repo OrderRepository // Injected dependency
}

func NewOrderService(repo OrderRepository) *OrderService {
    return &OrderService{repo: repo}
}

// ADAPTER (Implementation - Depends on infrastructure)
type PostgresRepo struct {
    db *sql.DB
}

func (p *PostgresRepo) Save(ctx context.Context, order Order) error {
    // SQL implementation details...
    return nil
}
```

## Interview Questions

### Q: What is the benefit of using Hexagonal Architecture (Ports and Adapters) in Go?
**A:** It decouples the core business logic from external concerns like the database, HTTP framework, or message queue. This makes the application easier to test (you can mock the ports), easy to swap technologies (e.g., switch from REST to gRPC handlers without touching business logic), and more resistant to "framework lock-in."

### Q: How do you handle Cross-Cutting Concerns (logging, metrics, auth) in Application Architecture?
**A:** In Go, we typically use the **Decorator Pattern** or **Middleware**.
-   **Middleware**: For HTTP layers (e.g., wrapping an `http.Handler`).
-   **Decorators**: For service layers. You can create a struct that embeds the service interface and adds logging before/after calling the underlying method.

```go
type LoggingMiddleware struct {
    next Service
}
func (l *LoggingMiddleware) DoWork() {
    log.Println("Starting...")
    l.next.DoWork()
    log.Println("Done.")
}
```

### Q: Explain the difference between "Domain Logic" and "Application Logic".
**A:** **Domain Logic** is business knowledge that is true regardless of the application (e.g., "A loan interest rate is 5%"). **Application Logic** (or Use Case) is the flow specific to the application (e.g., "Receive HTTP request, validate input, calculate interest, save to DB, send email"). Application logic orchestrates domain objects.
