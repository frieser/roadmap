---
---

## Summary
In Domain-Driven Design (DDD), an **Entity** is an object that is defined by a thread of continuity and identity, rather than by its attributes. Even if all its attributes change, it remains the same entity because of its unique identifier. Entities are mutable and typically have a lifecycle (created, updated, deleted).

## Detailed Explanation

### Core Characteristics
1.  **Identity**: The most distinguishing feature. Two entities are different if they have different IDs, even if all other fields are identical.
2.  **Mutability**: Entities change over time. Their state transitions should be managed via business methods (encapsulation), not public setters.
3.  **Lifecycle**: Entities are created, persist in the system, and may eventually be archived or deleted.

### Comparison with Value Objects
*   **Entities** matter because of *who* they are (ID).
*   **Value Objects** matter because of *what* they are (Values).

### Designing Entities
*   **Unique Identifier**: Use UUIDs, database sequences, or natural keys (e.g., SSN, Email - though risky if they change).
*   **Invariants**: Methods on the entity should ensure the object is always in a valid state.
*   **Behavior-Rich**: Avoid "anemic" entities that are just bags of data. Add logic to them.

## Go Example

```go
package domain

import (
	"errors"
	"time"
	"github.com/google/uuid"
)

// User is an Entity because it has a unique ID.
type User struct {
	id        uuid.UUID
	email     string
	isActive  bool
	createdAt time.Time
}

// NewUser acts as a factory/constructor ensuring initial validity
func NewUser(email string) (*User, error) {
	if email == "" {
		return nil, errors.New("email is required")
	}
	return &User{
		id:        uuid.New(),
		email:     email,
		isActive:  true,
		createdAt: time.Now(),
	}, nil
}

// ChangeEmail represents a state change in the entity's lifecycle
func (u *User) ChangeEmail(newEmail string) error {
	if newEmail == "" {
		return errors.New("invalid email")
	}
	u.email = newEmail
	return nil
}

// ID exposes the identity
func (u *User) ID() uuid.UUID {
	return u.id
}
```

## Interview Questions

### Q: How do you determine if a concept should be an Entity or a Value Object?
**A:** Ask if the object's identity matters. If you replace the object with another one having the same values, does the meaning change? If yes (e.g., a Person), it's an Entity. If no (e.g., a Color or Money), it's a Value Object.

### Q: Should Entities contain other Entities?
**A:** Yes, an Entity (specifically an Aggregate Root) can hold references to other Entities or Value Objects. However, it usually holds other Entities by ID to avoid loading huge object graphs.
