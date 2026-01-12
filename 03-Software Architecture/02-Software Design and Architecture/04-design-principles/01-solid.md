---
---

# SOLID Principles (Architectural Perspective)

## Summary
SOLID is a set of five design principles intended to make software designs more understandable, flexible, and maintainable. At the architectural level, these principles guide the decomposition of systems into decoupled, cohesive components.

## Detailed Explanation

### S: Single Responsibility Principle (SRP)
*   **Definition**: A component should have one, and only one, reason to change.
*   **Architectural Application**: Service boundaries. A service should own a single business capability. If a change in "Tax Calculation" requires redeploying the "Order Management" service, SRP is likely violated at the architectural level.

### O: Open/Closed Principle (OCP)
*   **Definition**: Software entities should be open for extension, but closed for modification.
*   **Architectural Application**: Plugin architectures. Use abstractions (interfaces/ports) to allow adding new features (adapters) without altering the core business logic.

### L: Liskov Substitution Principle (LSP)
*   **Definition**: Objects of a superclass should be replaceable with objects of its subclasses without breaking the application.
*   **Architectural Application**: Interface contracts. In a microservices environment, this translates to maintaining backward compatibility and adhering to strict API contracts.

### I: Interface Segregation Principle (ISP)
*   **Definition**: No client should be forced to depend on methods it does not use.
*   **Architectural Application**: Focused APIs. Avoid "God APIs". Use patterns like Backend-for-Frontend (BFF) to provide tailored interfaces for different clients (Mobile, Web, Admin).

### D: Dependency Inversion Principle (DIP)
*   **Definition**: High-level modules should not depend on low-level modules. Both should depend on abstractions.
*   **Architectural Application**: Hexagonal / Clean Architecture. The core domain should not know about the database, web server, or external APIs.

## Interview Questions
*   **Q: How does OCP apply to microservices?**
*   **A:** By using message queues and event-driven patterns, you can add new consumers (extension) without modifying the producer (core logic).
