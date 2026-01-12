---
---

## Summary
This note groups three fundamental concepts that often appear together in high-quality software engineering: **SOLID** (Design Principles), **TDD** (Development Methodology), and **DDD** (Architectural Approach). Mastering these separates a "coder" from a "software engineer."

## 1. SOLID Principles
A mnemonic for five design principles intended to make software designs more understandable, flexible, and maintainable.

### **S - Single Responsibility Principle (SRP)**
*   **Concept**: A class/module should have one, and only one, reason to change.
*   **Go Example**: Instead of a giant `User` struct that handles DB saving, validation, and email sending, split it into `UserRepository`, `UserValidator`, and `EmailService`.

### **O - Open/Closed Principle (OCP)**
*   **Concept**: Software entities should be open for extension, but closed for modification.
*   **Go Example**: Use interfaces. If you need to add a new Payment Method, implement the `Payer` interface rather than modifying the existing `ProcessPayment` switch statement.

### **L - Liskov Substitution Principle (LSP)**
*   **Concept**: Objects of a superclass shall be replaceable with objects of its subclasses without breaking the application.
*   **Go Example**: Since Go doesn't have inheritance, this applies to interfaces. If a function expects an `io.Reader`, passing a `*os.File` or a `*bytes.Buffer` should behave consistently.

### **I - Interface Segregation Principle (ISP)**
*   **Concept**: Clients should not be forced to depend upon interfaces that they do not use.
*   **Go Example**: "The bigger the interface, the weaker the abstraction." (Rob Pike). Use `io.Reader` (1 method) instead of `io.ReadWriteCloser` if you only need to read.

### **D - Dependency Inversion Principle (DIP)**
*   **Concept**: Depend upon abstractions, not concretions.
*   **Go Example**: Your business logic service should depend on a `Repository` interface, not a `*SQLDatabase` struct. This allows swapping SQL for MockDB during tests.

---

## 2. TDD (Test-Driven Development)
A development cycle where you write the test *before* the code.

### The Cycle: Red-Green-Refactor
1.  **Red**: Write a failing test that covers the desired functionality.
2.  **Green**: Write the minimum amount of code to make the test pass.
3.  **Refactor**: Clean up the code while ensuring the test still passes.

### Benefits for Architects
*   **Design Feedback**: TDD forces you to design testable APIs. If it's hard to test, it's bad architecture.
*   **Documentation**: Tests serve as "living documentation" of how the system should behave.
*   **Confidence**: Enables aggressive refactoring without fear of breaking regressions.

---

## 3. DDD (Domain-Driven Design)
An approach to software development that centers the design on the core domain and domain logic.

### Core Concepts
*   **Ubiquitous Language**: A shared vocabulary between developers and domain experts. If the expert says "Client," the code says `Client`, not `User` or `Customer`.
*   **Bounded Context**: The boundary within which a specific domain model is defined and applicable. (e.g., "Product" means something different in "Sales Context" vs "Shipping Context").
*   **Entities**: Objects defined by their identity (e.g., User ID). Mutable.
*   **Value Objects**: Objects defined by their attributes (e.g., Color, Money). Immutable.
*   **Aggregates**: A cluster of domain objects that can be treated as a single unit. Accessed only through the **Aggregate Root**.

---

## Go Application Example (Combining Principles)

```go
package domain

// DDD: Value Object (Immutable)
type Money struct {
    amount   float64
    currency string
}

// DDD: Entity (Has Identity)
type Order struct {
    ID    string
    Total Money
    Status string
}

// SOLID: DIP & ISP - Interface defined in the domain
type OrderRepository interface {
    Save(o *Order) error
}

// SOLID: SRP - Service handles logic, not DB
type OrderService struct {
    repo OrderRepository // Dependency Injection
}

func (s *OrderService) PlaceOrder(o *Order) error {
    // TDD: This logic would be written after a failing test
    if o.Total.amount <= 0 {
        return fmt.Errorf("invalid order amount")
    }
    return s.repo.Save(o)
}
```

## Interview Questions

**Q: Explain the Dependency Inversion Principle. Is it the same as Dependency Injection?**
**A:** No. **Dependency Inversion** is the *principle* (High-level modules should not depend on low-level modules; both should depend on abstractions). **Dependency Injection** is a *pattern* used to implement that principle (passing the dependencies into the object rather than creating them inside).

**Q: In DDD, what is an Aggregate Root?**
**A:** It is the main Entity that acts as the gateway to a cluster of related objects (the Aggregate). External objects are only allowed to hold references to the Root, not to the internal members. This ensures data integrity and transaction boundaries.

**Q: How does TDD influence Software Architecture?**
**A:** TDD encourages **Testability** as a first-class architectural concern. It naturally leads to loosely coupled components (to allow mocking) and highly cohesive interfaces (to make testing specific behaviors easier).
