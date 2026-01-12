---
---

## Summary
**Architectural Patterns** define the high-level structure and organization of software systems. Unlike design patterns (which solve localized coding problems), architectural patterns address system-wide concerns like scalability, maintainability, and data flow. In Go, **Clean/Hexagonal Architecture** is the dominant standard for backend services.

## Detailed Explanation

### 1. Layered Architecture (N-Tier)
The most traditional pattern. Code is organized into horizontal layers, each with a specific responsibility.
*   **Layers**: Presentation (HTTP) -> Business Logic (Service) -> Data Access (Repository) -> Database.
*   **Pros**: Simple to understand, separation of concerns.
*   **Cons**: Can lead to "tight coupling" if layers bypass each other; often database-centric.

### 2. Clean / Hexagonal Architecture (Ports and Adapters)
Heavily widely adopted in the Go community (popularized by Uncle Bob). It inverts dependencies so that **business logic depends on nothing**.
*   **Core (Entities)**: Pure business rules. No dependencies.
*   **Use Cases (Services)**: Application logic. Depends only on Core.
*   **Adapters (Interface Adapters)**: HTTP handlers, SQL Repositories. Depend on Use Cases.
*   **Dependency Rule**: Source code dependencies can only point **inwards**.
*   **Pros**: Highly testable (business logic isolated), framework independent, database independent.

### 3. Microservices Architecture
Decomposing an application into loosely coupled services, usually communicating via HTTP/gRPC.
*   **Pros**: Independent scaling, technology diversity, fault isolation.
*   **Cons**: Distributed system complexity (latency, consistency, observability).

### 4. Event-Driven Architecture (EDA)
Services communicate by emitting and consuming **events** (messages) asynchronously via a broker (Kafka, RabbitMQ).
*   **Pros**: Decoupling, scalability, responsiveness.
*   **Cons**: Harder to debug flow, eventual consistency challenges.

## Go Example: Hexagonal Structure

A typical Go project structure following Hexagonal/Clean principles:

```text
/cmd
  /api
    main.go            # Entry point, dependency wiring
/internal
  /core
    /domain            # Enterprise business rules (Entities)
      user.go
    /ports             # Interfaces (Ports)
      user_repo.go     # Repository Interface
      user_service.go  # Service Interface
  /service             # Application business rules (Use Cases)
    user_service.go    # Implements Service Interface
  /adapters
    /handler           # Driving Adapter (HTTP)
      user_http.go
    /repository        # Driven Adapter (Database)
      user_postgres.go # Implements Repository Interface
```

## Go Code Snippet (Dependency Inversion)

```go
package main

// --- Domain/Port Layer ---
type User struct {
	ID   int
	Name string
}

// Repository Interface (Port) - Logic doesn't know about SQL
type UserRepository interface {
	Save(u *User) error
}

// --- Service Layer ---
type UserService struct {
	repo UserRepository // Depends on interface, not implementation
}

func NewUserService(r UserRepository) *UserService {
	return &UserService{repo: r}
}

func (s *UserService) CreateUser(name string) error {
	user := &User{Name: name}
	return s.repo.Save(user) // Logic remains pure
}

// --- Adapter Layer ---
type PostgresRepo struct {
	// db *sql.DB
}

func (r *PostgresRepo) Save(u *User) error {
	// SQL implementation details...
	return nil
}

// --- Wiring (Main) ---
func main() {
	repo := &PostgresRepo{}       // Concrete Adapter
	svc := NewUserService(repo)   // Inject into Service
	svc.CreateUser("Alice")
}
```

## Interview Questions

### Q: What is the main benefit of Hexagonal Architecture over Layered Architecture?
**A:** **Dependency Inversion**. In Layered Architecture, the Business Logic often depends on the Database layer. In Hexagonal, the Business Logic defines an **interface** (Port) that the Database layer must implement (Adapter). This makes the Business Logic independent of the database, allowing for easier testing (mocking) and swapping of infrastructure.

### Q: When should you chose Monolith over Microservices in Go?
**A:** Start with a **Modular Monolith**. Go's strong typing and package system make it excellent for building large, structured monoliths. Only switch to Microservices when specific organizational scaling needs (independent teams deploying independently) or technical scaling needs (cpu-intensive parts needing separate scaling) justify the massive operational complexity overhead.

### Q: What is the Role of `internal` package in Go architecture?
**A:** The `internal` directory enforces encapsulation at the project level. Code inside `internal` cannot be imported by other projects/modules. This is used to hide the implementation details of your architecture (Adapters, Services) while exposing only the necessary API.
