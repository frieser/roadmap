---
---

# Layered Architecture (N-Tier)

## Summary
Layered Architecture (often called N-Tier) is the most common and traditional architectural style. It organizes code into horizontal layers, where each layer performs a specific role and only communicates with adjacent layers. This separation of concerns simplifies development, testing, and maintenance, making it a standard starting point for many enterprise applications.

## Detailed Explanation

### 1. Standard Layers

In a typical 4-layer architecture:

1.  **Presentation Layer (UI)**: Handles user interaction and JSON/HTML rendering. It should not contain business logic.
2.  **Business Logic Layer (Service)**: The core of the application. Encapsulates rules, validations, and calculations.
3.  **Data Access Layer (Repository/DAO)**: Abducts the database interaction (SQL queries, ORM calls).
4.  **Database Layer**: The physical storage (PostgreSQL, MySQL).

### 2. Communication Rules

*   **Strict (Closed) Layers**: A layer can *only* call the layer immediately below it. (Presentation -> Business -> Data).
    *   *Pro*: Strict isolation. Changing the DB schema only affects the Data layer.
    *   *Con*: "Sinkhole" anti-pattern—requests just pass through layers without logic, adding boilerplate.
*   **Relaxed (Open) Layers**: A layer can call any layer below it (e.g., Presentation -> Data).
    *   *Pro*: Performance, less boilerplate.
    *   *Con*: High coupling. Harder to test.

### 3. Pros & Cons

| Feature | Description |
| :--- | :--- |
| **Simplicity** | Easy to understand and organize teams around (UI team, Backend team). |
| **Testability** | Each layer can be mocked and tested independently. |
| **Separation of Concerns** | UI logic doesn't leak into SQL queries. |
| **Scalability** | Limited. The entire monolith usually scales together (unless tiers are physically separated). |
| **Performance** | Can be slower due to multiple hops through layers. |

## Relationship with MVC
Layered Architecture is often confused with **MVC (Model-View-Controller)**.
*   **MVC** is a *design pattern* primarily for the Presentation Layer.
*   **Layered Architecture** is a system-wide *architectural style*.
*   In a layered app, the "Controller" belongs to the Presentation Layer, the "Model" often spans Business/Data layers, and the "View" is the UI output.

## Go Implementation Example

In Go, layers are typically represented by packages/structs with interfaces to decouple them.

```go
// DIRECTORY STRUCTURE
// ├── handler/      (Presentation Layer)
// ├── service/      (Business Logic Layer)
// ├── repository/   (Data Access Layer)
// └── model/        (Domain Objects)

// --- REPOSITORY LAYER ---
type UserRepo interface {
    Save(user User) error
}

type PostgresRepo struct { db *sql.DB }

func (r *PostgresRepo) Save(u User) error {
    _, err := r.db.Exec("INSERT INTO users...", u.Name)
    return err
}

// --- SERVICE LAYER ---
type UserService struct {
    repo UserRepo // Dependency Injection
}

func (s *UserService) RegisterUser(name string) error {
    if name == "" {
        return errors.New("name is empty") // Business Rule
    }
    user := User{Name: name}
    return s.repo.Save(user)
}

// --- HANDLER LAYER ---
type UserHandler struct {
    svc *UserService
}

func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    name := r.FormValue("name")
    if err := h.svc.RegisterUser(name); err != nil {
        http.Error(w, err.Error(), 400)
        return
    }
    w.WriteHeader(201)
}
```

## Interview Questions

**Q: What is the "Sinkhole" anti-pattern in Layered Architecture?**
**A:** It occurs when requests simply pass through multiple layers (e.g., Service layer just calls Repository layer) without performing any logic. This adds unnecessary complexity and latency. It's often solved by allowing "Open Layers" for simple read operations.

**Q: How does Layered Architecture differ from Hexagonal (Clean) Architecture?**
**A:** Layered Architecture is based on *dependencies pointing down* (UI -> Logic -> Data). The Database is often the foundation.
Hexagonal Architecture uses *Dependency Inversion* so dependencies point *inward*. The Domain Logic is the center, and both the UI and Database are plugins (adapters) to it.

**Q: Can you scale a Layered Architecture?**
**A:** Yes, but typically by scaling the entire application (Horizontal Scaling of the monolith). You can also physically separate tiers (e.g., App Server on one machine, DB on another), but it's less flexible than Microservices scaling.
