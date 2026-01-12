---
---

## Summary
In **Domain-Driven Design (DDD)**, the **Domain Model** is the heart of the application. It represents the business logic, rules, and data. A key architectural choice is between a **Rich Domain Model** (logic and data combined) and an **Anemic Domain Model** (data only, logic in services).

## Detailed Explanation

### Rich Domain Model
A rich domain model is an object-oriented approach where domain objects (Entities and Value Objects) contain both data and the logic that operates on that data.
*   **Encapsulation**: State transitions are controlled via methods that enforce business rules.
*   **Maintainability**: Logic is located where the data lives, making it easier to find and test.
*   **Alignment with OOP**: Follows the core principle that objects should have behavior.

### Anemic Domain Model
An anemic model treats domain objects as simple data structures (POJOs/DTOs) with only getters and setters. All business logic is moved into "Service" classes.
*   **The "Anti-Pattern" View**: Martin Fowler describes this as an anti-pattern because it defeats the purpose of object-oriented design, essentially reverting to procedural programming.
*   **When it's used**: Often seen in simple CRUD applications where there is little to no complex business logic.

### Comparison Table

| Feature | Rich Domain Model | Anemic Domain Model |
| :--- | :--- | :--- |
| **Logic Location** | Inside Entities/Value Objects | Inside Service Layers |
| **Data Protection** | Enforced via Private Fields/Methods | Open (Public Getters/Setters) |
| **Complexity** | Best for high complexity | Suitable for low complexity (CRUD) |
| **Testability** | Unit test entities directly | Test services (requires mocking data) |

## Go Example: Enforcing Invariants

```go
package domain

import "errors"

// Order represents a Rich Domain Model entity
type Order struct {
	id     string
	items  []string
	status string
}

func NewOrder(id string) *Order {
	return &Order{id: id, status: "PENDING"}
}

// AddItem enforces business rules (invariants)
func (o *Order) AddItem(item string) error {
	if o.status != "PENDING" {
		return errors.New("cannot add items to a non-pending order")
	}
	o.items = append(o.items, item)
	return nil
}

func (o *Order) Status() string {
	return o.status
}
```

## Interview Questions

### Q: Why is an Anemic Domain Model considered an anti-pattern?
**A:** Because it strips objects of their behavior, leading to a procedural design where logic is separated from data. This makes the system harder to maintain as business rules grow complex and fragmented across multiple services.

### Q: How do you transition from an Anemic to a Rich model?
**A:** By identifying business logic in services that operates on an entity's data and moving that logic into methods on the entity itself, while making the entity's fields private to ensure they can only be modified through those methods.
