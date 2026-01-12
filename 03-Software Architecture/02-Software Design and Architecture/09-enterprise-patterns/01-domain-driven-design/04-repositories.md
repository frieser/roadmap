---
---

## Summary
A **Repository** is a mechanism for encapsulating storage, retrieval, and search behavior which emulates a collection of objects. It mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.

## Detailed Explanation

### Core Concepts
1.  **Collection Abstraction**: To the client (Domain Service), a Repository looks like an in-memory collection (List, Map). You `Add` or `Remove` items.
2.  **Decoupling**: It hides the details of data access (SQL, NoSQL, File System). The domain model doesn't know *how* data is saved, only *that* it is saved.
3.  **Aggregate Focus**: Repositories should only be created for **Aggregate Roots**. If you need an item inside an aggregate (e.g., an OrderLine), you get it through the Order Repository.

### Repository vs DAO (Data Access Object)
*   **DAO**: Closer to the database table. Often one DAO per table. Focuses on CRUD.
*   **Repository**: Closer to the domain. One Repository per Aggregate Root. Focuses on business intent (e.g., `FindEligibleUsers` vs `Select * Where status=1`).

## Go Example

```go
package domain

import (
	"context"
	"errors"
	"github.com/google/uuid"
)

var ErrNotFound = errors.New("entity not found")

// UserRepository is the interface defined in the DOMAIN layer.
// Implementation (SQL, Postgres) belongs in the INFRASTRUCTURE layer.
type UserRepository interface {
	Save(ctx context.Context, user *User) error
	FindByID(ctx context.Context, id uuid.UUID) (*User, error)
	Delete(ctx context.Context, id uuid.UUID) error
}

// InMemoryUserRepo is a simple implementation for testing/prototyping
type InMemoryUserRepo struct {
	store map[uuid.UUID]*User
}

func NewInMemoryUserRepo() *InMemoryUserRepo {
	return &InMemoryUserRepo{
		store: make(map[uuid.UUID]*User),
	}
}

func (r *InMemoryUserRepo) Save(ctx context.Context, user *User) error {
	r.store[user.ID()] = user
	return nil
}

func (r *InMemoryUserRepo) FindByID(ctx context.Context, id uuid.UUID) (*User, error) {
	u, ok := r.store[id]
	if !ok {
		return nil, ErrNotFound
	}
	return u, nil
}
```

## Interview Questions

### Q: Should you have a Repository for every domain object?
**A:** No. You should only have Repositories for **Aggregate Roots**. Child entities and value objects should be accessed and persisted via the Aggregate Root's repository to ensure consistency and transaction boundaries.

### Q: Where should the Repository interface and implementation reside?
**A:** The **Interface** belongs in the Domain Layer (Hexagonal/Clean Architecture), while the **Implementation** belongs in the Infrastructure Layer. This allows the domain to depend on abstractions, not concretions (Dependency Inversion Principle).
