---
---

## Summary
Object-Oriented Programming (OOP) remains a cornerstone of software architecture, providing the mental model for organizing complex systems into manageable, encapsulated units. In modern cloud-native and microservices environments, OOP principles have evolved: strict class hierarchies are often replaced by **Composition over Inheritance**, and service boundaries frequently mirror class boundaries (encapsulation at scale). For an architect, OOP is less about "classes" and more about **managing state, behavior, and dependencies** to ensure long-term maintainability.

## Detailed Explanation

### 1. OOP in Modern Architecture (Microservices & Cloud-native)
While the industry has seen a rise in Functional Programming (FP) and Data-Oriented Design, OOP principles remain vital for high-level system design:
*   **Encapsulation at Scale**: A microservice can be viewed as an "object" in a distributed system. It encapsulates its data (database) and exposes behavior through a well-defined interface (API).
*   **Single Responsibility (SRP)**: Applied at the service level, this prevents the creation of "God Services" and ensures each component has a clear, singular purpose.
*   **Polymorphism through Interfaces**: Modern architectures use interfaces (gRPC, OpenAPI) to allow multiple implementations of a service to exist, facilitating A/B testing, canary releases, and versioning.

### 2. Composition over Inheritance
Modern software design strongly favors **Composition** (the "Has-A" relationship) over **Inheritance** (the "Is-A" relationship).
*   **The Problem with Inheritance**: Deep inheritance hierarchies lead to the "Fragile Base Class" problem, where changes to a parent class cause unexpected breakage in many children. It also leads to tight coupling.
*   **The Benefit of Composition**: By combining small, independent objects/components, architects create more flexible systems. It allows for behavior to be changed at runtime (Strategy pattern) and simplifies testing through easy mocking.
*   **Modern Languages**: Languages like **Go** and **Rust** don't support class-based inheritance at all, forcing developers to use composition and interfaces—a design choice reflecting modern architectural best practices.

### 3. Key Design Patterns for Architects
Three patterns from the Gang of Four (GoF) are particularly relevant for modern system architecture:

| Pattern | Architectural Use Case | Why it Matters |
| :--- | :--- | :--- |
| **Strategy** | Switching between cloud providers, payment gateways, or AI models. | Decouples business logic from specific implementations. |
| **Factory** | Dependency Injection (DI) and dynamic service provisioning. | Centralizes object creation, making the system more testable and configurable. |
| **Adapter** | Hexagonal Architecture (Ports and Adapters). | Bridges the gap between core business logic and external infrastructure (DBs, third-party APIs). |

### 4. Anemic vs. Rich Domain Model
This is a critical architectural decision in Domain-Driven Design (DDD).

*   **Anemic Domain Model**: Entities are mere data containers (getters/setters). Business logic lives in "Service" classes.
    *   *Pros*: Simple for CRUD applications; easy to understand initially.
    *   *Cons*: Business logic becomes fragmented and hard to maintain as complexity grows; violates encapsulation.
*   **Rich Domain Model**: Entities encapsulate both state and behavior. Business rules (invariants) are enforced within the entity.
    *   *Pros*: Logic is centralized and consistent; prevents invalid states; aligns with true OOP principles.
    *   *Cons*: Steeper learning curve; requires careful design to avoid bloated entities.

## Go Application Examples

### Strategy Pattern in Go
Using interfaces and composition to switch implementations at runtime.

```go
package main

import "fmt"

// PaymentStrategy defines the interface for different payment methods
type PaymentStrategy interface {
	Pay(amount float64) error
}

// CreditCard implementation
type CreditCard struct {
	CardNumber string
}

func (c *CreditCard) Pay(amount float64) error {
	fmt.Printf("Paid $%.2f using Credit Card: %s\n", amount, c.CardNumber)
	return nil
}

// PayPal implementation
type PayPal struct {
	Email string
}

func (p *PayPal) Pay(amount float64) error {
	fmt.Printf("Paid $%.2f using PayPal: %s\n", amount, p.Email)
	return nil
}

// Checkout uses the strategy
type Checkout struct {
	Strategy PaymentStrategy
}

func (c *Checkout) Process(amount float64) {
	c.Strategy.Pay(amount)
}

func main() {
	cart := &Checkout{Strategy: &CreditCard{CardNumber: "1234-5678"}}
	cart.Process(100.00)

	// Change strategy at runtime
	cart.Strategy = &PayPal{Email: "user@example.com"}
	cart.Process(50.00)
}
```

### Rich Domain Model vs Anemic (Go approach)
In Go, we use methods on structs to enforce invariants.

```go
// RICH MODEL (Recommended)
type Account struct {
	balance float64 // unexported to enforce encapsulation
}

func (a *Account) Deposit(amount float64) error {
	if amount <= 0 {
		return fmt.Errorf("deposit must be positive")
	}
	a.balance += amount
	return nil
}

func (a *Account) Balance() float64 {
	return a.balance
}

// ANEMIC MODEL (Avoid for complex logic)
type AnemicAccount struct {
	Balance float64 // Exported field, anyone can modify it directly
}
```

## Interview Questions

### Q: Why is "Composition over Inheritance" preferred in modern architecture?
**A:** Composition provides greater flexibility and looser coupling. It avoids the "fragile base class" problem where changes in a parent ripple down unpredictably. It also makes testing easier by allowing components to be mocked individually through interfaces.

### Q: How does a Rich Domain Model help in Domain-Driven Design?
**A:** It ensures that business logic and data stay together. By encapsulating rules (invariants) within entities, you prevent the application from entering an invalid state and make the code "self-documenting" regarding business requirements.

### Q: How would you implement the Adapter pattern in a Hexagonal Architecture?
**A:** You define an interface (Port) in the core domain layer. Then, you create a concrete implementation (Adapter) in the infrastructure layer that "wraps" the external tool (e.g., a specific database driver) to match the domain's interface.

### Q: Can OOP and Functional Programming (FP) coexist in an architecture?
**A:** Yes, and they often should. A common pattern is using OOP for high-level structure and state management (Encapsulation), while using FP principles (pure functions, immutability) for complex algorithmic logic within methods.
