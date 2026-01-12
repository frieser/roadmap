# Layered Architecture (N-Tier)

## Summary
Layered Architecture describes a system partitioned into horizontal layers, where each layer performs a specific role within the application (e.g., presentation logic vs. business logic). It is the most common pattern for enterprise applications due to its simplicity, separation of concerns, and testability.

## Detailed Explanation

### 1. Standard Layers
While the number of layers can vary, the classic model consists of four:

1.  **Presentation Layer**: Responsible for the UI and handling user interactions. In a Web API, this is the Controller layer that parses JSON/HTTP.
2.  **Business Layer (Service)**: Implements the core business logic, rules, and workflows. It should be agnostic of the UI or Database.
3.  **Persistence Layer (Data Access)**: Abstracts the storage mechanism. Contains Repositories/DAOs that speak SQL or NoSQL.
4.  **Database Layer**: The actual underlying data store (PostgreSQL, Mongo, etc.).

### 2. Closed vs. Open Layers
*   **Closed (Strict)**: A layer can only call the layer immediately below it. (Presentation -> Business -> Persistence).
    *   *Pros*: High isolation, easy to refactor/replace layers.
    *   *Cons*: Can lead to "proxy" methods that just pass data along.
*   **Open (Relaxed)**: A layer can skip the one below it (e.g., Presentation calling Persistence directly for a simple read).
    *   *Pros*: Performance, less boilerplate.
    *   *Cons*: Tighter coupling, spaghetti dependencies.

### 3. The Sinkhole Anti-Pattern
This occurs when requests simply pass through multiple layers without any logic being applied.
*   *Example*: Controller calls Service.getAll() -> Service calls Repo.getAll() -> Repo calls DB.
*   *Impact*: Unnecessary complexity and code noise.
*   *Solution*: If this is the norm (>20%), consider a simpler architecture (e.g., Transaction Script) or opening up the layers for read operations.

### 4. Comparison with Other Patterns
*   **vs. Hexagonal**: Layered is **Top-Down** (UI depends on Domain, Domain depends on DB). Hexagonal is **Center-Out** (UI and DB depend on Domain). Hexagonal solves the problem of the Domain becoming coupled to the Database.
*   **vs. MVC**: MVC is primarily a pattern for the **Presentation Layer** itself, not necessarily the whole backend architecture.

## Go Implementation (Standard 3-Layer)

```go
package main

// --- Persistence Layer ---
type UserRepository struct{}

func (r *UserRepository) GetByID(id int) string {
	return "UserFromDB"
}

// --- Business Layer ---
type UserService struct {
	repo *UserRepository
}

func (s *UserService) GetUser(id int) string {
	// Business logic could go here (e.g., logging, validation)
	return s.repo.GetByID(id)
}

// --- Presentation Layer ---
type UserController struct {
	service *UserService
}

func (c *UserController) HandleRequest(id int) {
	user := c.service.GetUser(id)
	println("Response:", user)
}

func main() {
	// Wiring dependencies (Dependency Injection)
	repo := &UserRepository{}
	service := &UserService{repo: repo}
	controller := &UserController{service: service}

	controller.HandleRequest(1)
}
```

## Interview Questions

*   **Q: What is the main disadvantage of Layered Architecture?**
    *   **A:** It tends to lead to **Database-Driven Design** rather than Domain-Driven Design. Since the Business layer depends on the Persistence layer, changes to the database schema often ripple up and force changes in the business logic. Hexagonal Architecture fixes this by inverting the dependency.
*   **Q: How do you prevent "spaghetti code" in a Layered Architecture?**
    *   **A:** Enforce strict **unidirectional dependencies** (top-down). Using linting tools or architectural unit tests (e.g., ArchUnit) to forbid upper layers from being imported by lower layers.
*   **Q: Is it acceptable for the Presentation Layer to access the Database directly?**
    *   **A:** Generally No. This creates tight coupling and scatters SQL/Data logic across the UI code, making the system hard to test and unsecure. It violates the Separation of Concerns.
