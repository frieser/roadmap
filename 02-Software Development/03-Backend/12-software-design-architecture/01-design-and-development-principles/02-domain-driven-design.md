---
---

## Summary
Domain-Driven Design (DDD) is an approach to software development that focuses on complex domains by connecting the implementation to an evolving model of the core business concepts. In Go, DDD is implemented using a layered architecture, where business logic is isolated from technical details like databases or APIs.

## Detailed Explanation

DDD consists of two types of patterns: Strategic and Tactical.

### Strategic Patterns
*   **Bounded Context**: A central pattern in DDD. It defines a boundary (typically a subsystem or a team's work) within which a particular model is defined and applicable.
*   **Ubiquitous Language**: A common language shared by everyone on the team (developers, domain experts, business analysts) to describe the domain model.
*   **Context Mapping**: Defines how different Bounded Contexts relate to each other (e.g., Shared Kernel, Customer/Supplier).

### Tactical Patterns
*   **Entities**: Objects that have a unique identity that persists over time and across different states.
*   **Value Objects**: Objects that describe characteristics and have no identity (e.g., an Address or Money). They are immutable.
*   **Aggregates**: A cluster of domain objects that can be treated as a single unit. Every aggregate has a "Root" entity.
*   **Repositories**: Interfaces used to recover and persist aggregates.
*   **Domain Services**: Contain business logic that doesn't naturally belong to an entity or value object.

## Go-specific Context and Examples

In Go, DDD often leads to a "Hexagonal" or "Clean" architecture. Business rules are placed in an internal package, and dependencies point inwards.

### Domain Entity and Value Object
```go
package domain

import (
	"errors"
	"github.com/google/uuid"
)

// Money is a Value Object (Immutable)
type Money struct {
	amount   float64
	currency string
}

func NewMoney(amount float64, currency string) (Money, error) {
	if amount < 0 {
		return Money{}, errors.New("amount cannot be negative")
	}
	return Money{amount: amount, currency: currency}, nil
}

// User is an Entity (Has ID)
type User struct {
	ID    uuid.UUID
	Name  string
	Email string
}
```

### Aggregate Root and Repository Interface
In Go, repositories are defined as interfaces in the domain layer, and implemented in the infrastructure layer.

```go
package domain

import "context"

// Order is an Aggregate Root
type Order struct {
	ID    uuid.UUID
	Items []OrderItem
	Total Money
}

// OrderRepository is the interface for persistence
type OrderRepository interface {
	Save(ctx context.Context, order *Order) error
	GetByID(ctx context.Context, id uuid.UUID) (*Order, error)
}
```

### Domain Service
```go
package domain

// DiscountService is a Domain Service
type DiscountService struct{}

func (s *DiscountService) CalculateDiscount(order *Order) Money {
	// Business logic to calculate discount
	return Money{amount: 5.0, currency: "USD"}
}
```

## Interview Questions

**Q: What is the difference between an Entity and a Value Object in DDD?**
**A:** An Entity is defined by its identity (e.g., a User with a unique ID), while a Value Object is defined by its attributes (e.g., Money with amount and currency). If two Value Objects have the same attributes, they are considered equal; two Entities are only equal if they have the same ID.

**Q: How do you handle cross-aggregate operations in DDD?**
**A:** Cross-aggregate operations should typically be handled by Domain Services or through eventual consistency using Domain Events. You should avoid modifying multiple aggregates in a single transaction to maintain clear boundaries.

**Q: Why is the Ubiquitous Language important in DDD?**
**A:** It reduces translation overhead and misunderstandings between technical and non-technical team members. By using the same terms in the code as in business discussions, the code becomes a living representation of the business domain.
