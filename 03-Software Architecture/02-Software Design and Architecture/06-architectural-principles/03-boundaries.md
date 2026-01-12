---
---

## Summary
**Boundaries** (Architectural Boundaries) are the lines that separate software elements from one another. They separate the high-level policy (Business Rules) from low-level details (Plugins, DBs, UI). A good architect creates boundaries to minimize the cost of change by preventing the propagation of changes across the boundary.

## Detailed Explanation

### 1. The Plugin Architecture
The ultimate goal of boundaries is to create a **Plugin Architecture**. The Core Business Logic is the "Host", and everything else (DB, Web, 3rd Party APIs) are "Plugins".
*   *Benefit*: You can swap, update, or deploy plugins independently of the core.

### 2. Crossing the Boundary
Boundaries are crossed using **Interfaces** and **Data Transfer Objects (DTOs)**.
*   **Control Flow**: Usually points from High Level to Low Level (Controller calls Interactor).
*   **Source Dependency**: MUST point against the control flow (Inversion of Control). The Interactor defines an Interface, and the Controller (or Presenter) implements it.
*   **Data**: Data crossing the boundary should be simple structures (DTOs), not Entity objects (which might contain methods/logic) or DB Rows (which are coupled to the schema).

### 3. Partial Boundaries
Sometimes a full architectural boundary (Interfaces + DTOs + Separate Compilation Units) is too expensive (YAGNI).
*   **Strategy**: Use "Partial Boundaries" like defining the Interface but keeping the implementation in the same module, or using a Facade. This reserves the *option* to create a full boundary later.

## Go Application (The Input/Output Port)

In Clean Architecture, boundaries are explicit "Ports".

```go
// --- CORE BOUNDARY (The Line) ---

// Input Port: How the world talks to the Core
type CreateUserUseCase interface {
    Execute(req CreateUserRequest) (*CreateUserResponse, error)
}

// Output Port: How the Core talks to the world (DB)
type UserRepository interface {
    Save(u User) error
}

// DTOs (Simple data, no logic)
type CreateUserRequest struct {
    Email string
}
type CreateUserResponse struct {
    ID string
}

// --- CROSSING THE BOUNDARY ---

// Implementation (Inside Core)
type UserInteractor struct {
    Repo UserRepository
}

func (i *UserInteractor) Execute(req CreateUserRequest) (*CreateUserResponse, error) {
    // Logic...
    i.Repo.Save(user) // Crossing Output Boundary
    return &CreateUserResponse{ID: user.ID}, nil
}
```

## Interview Questions

**Q: Why shouldn't you pass a Database Row object (ORM Entity) across a boundary into the UI?**
**A:** That creates a dependency from the UI directly to the Database Schema. If you rename a column in the DB, the ORM object changes, and the UI breaks. This violates the boundary. You should map the ORM object to a Response DTO inside the boundary, protecting the UI from DB changes.

**Q: What is the difference between a "Layer" and a "Boundary"?**
**A:** Layers are a way to organize code (horizontal slicing). Boundaries are separation lines that enforce dependency rules (vertical or horizontal). A Layered Architecture (UI -> Service -> DAO) has boundaries, but often they are "soft" (leaky). A strict Architectural Boundary (like in Hexagonal) uses Dependency Inversion to ensure dependencies point strictly one way.

**Q: When is a boundary "too expensive"?**
**A:** Every boundary requires:
1.  Defining Interfaces.
2.  Creating DTOs (mapping data back and forth).
3.  Dependency Injection setup.
If the application is a simple CRUD script, the cost of this boilerplate outweighs the benefit of decoupling. Boundaries pay off in *complex, long-lived* systems.
