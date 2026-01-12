---
---

## Summary
**Coupling** and **Cohesion** are the yin and yang of software design.
*   **Coupling**: The degree of interdependence between software modules. (Goal: **Low**)
*   **Cohesion**: The degree to which elements inside a module belong together. (Goal: **High**)
High Cohesion and Low Coupling produce systems that are easy to maintain, test, and extend.

## Detailed Explanation

### 1. Coupling (The glue between modules)
*   **Tight Coupling**: Module A knows internal details of Module B. Changing B breaks A.
    *   *Causes*: Direct database access, shared global variables, inheriting implementation.
*   **Loose Coupling**: Module A talks to Module B via a stable interface/contract. B can be rewritten completely as long as the contract holds.
    *   *Enablers*: Dependency Injection, Interfaces, Event Busses.

### 2. Cohesion (The glue within a module)
*   **Low Cohesion (Random)**: A "Utils" class with `CalculateTax()`, `SendEmail()`, and `ResizeImage()`. These don't belong together.
*   **High Cohesion (Functional)**: A `ImageProcessor` module where every function relates to image manipulation.
*   **SRP Connection**: High cohesion usually means adhering to the Single Responsibility Principle.

### 3. The Tension
There is a trade-off. To minimize coupling (make everything independent), you might split things too much, losing cohesion (related logic is scattered). To maximize cohesion (put everything related in one place), you might create a God Class that is coupled to everything. The art of architecture is finding the balance.

## Go Application

### Low Cohesion / Tight Coupling (Bad)
A `UserHandler` that handles HTTP, DB, and Business Logic.

```go
type UserHandler struct {
    db *sql.DB
}

func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    // 1. Parse JSON (Transport Layer)
    var u User
    json.NewDecoder(r.Body).Decode(&u)
    
    // 2. Validate (Business Logic) - Should be Cohesive in a Domain Service
    if len(u.Password) < 8 { http.Error(...) }
    
    // 3. SQL (Persistence Layer) - Coupled to SQL
    h.db.Exec("INSERT INTO users...", u.Name)
}
```

### High Cohesion / Loose Coupling (Good)
Separation of concerns.

```go
// Cohesive: Only handles HTTP
func (h *UserHandler) CreateUser(...) {
    req := parse(r)
    // Decoupled: Calls interface
    err := h.service.Register(req) 
    writeResponse(w, err)
}

// Cohesive: Only handles Business Rules
func (s *UserService) Register(u User) error {
    if !s.validator.IsValid(u) { return ErrorInvalid }
    // Decoupled: Calls interface
    return s.repo.Save(u)
}
```

## Interview Questions

**Q: Can you have too much decoupling?**
**A:** Yes. It's called the "Poltergeist" or "Gas Factory" anti-pattern. If you break a simple task into 20 tiny micro-classes that just pass data to each other without doing work, you've increased complexity (cognitive load) without gaining flexibility. You've made the code hard to navigate.

**Q: What is "Connascence"?**
**A:** It's a metric for coupling. Two components are connascent if a change in one requires a change in the other to maintain correctness.
*   *Connascence of Name*: Changing a function name (Low coupling, easy refactor).
*   *Connascence of Meaning*: `int status = 1` means "Active". Both sides must know `1` is "Active". (Higher coupling).
*   *Connascence of Algorithm*: Both sides must use the exact same hashing algorithm (High coupling).

**Q: How do Microservices impact Coupling and Cohesion?**
**A:** 
*   **Cohesion**: Microservices force high cohesion by grouping logic around a Bounded Context.
*   **Coupling**: They enforce loose coupling via network boundaries (API contracts). However, they introduce "Temporal Coupling" (Service A needs Service B to be online).
