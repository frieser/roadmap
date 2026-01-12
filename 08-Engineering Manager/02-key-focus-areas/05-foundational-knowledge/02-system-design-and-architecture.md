## Summary
System design and architecture involve defining the elements of a system like modules, architecture, components, and their interfaces and data for a system to satisfy specified requirements. It requires balancing trade-offs between consistency, availability, and partition tolerance (CAP theorem) while ensuring scalability and maintainability.

## Detailed Explanation
### Key Architectural Patterns
*   **Monolithic**: Single codebase, easy to develop initially but hard to scale.
*   **Microservices**: Loosely coupled services, independently deployable, complex to manage.
*   **Event-Driven**: Components communicate via events, promoting decoupling (e.g., Kafka, RabbitMQ).
*   **Serverless**: Execution model where the cloud provider manages the server allocation.

### Design Principles
*   **SOLID**: Five principles for object-oriented design to make software more understandable and flexible.
*   **DRY (Don't Repeat Yourself)**: Reducing repetition of software patterns.
*   **KISS (Keep It Simple, Stupid)**: Systems work best if they are kept simple.

### Go Code Example: Clean Architecture
Go's implicit interface implementation makes it excellent for Clean Architecture, where business logic is decoupled from external concerns (database, UI).

```go
package main

import "fmt"

// --- Domain Layer (Core Business Logic) ---

// User entity
type User struct {
	ID   int
	Name string
}

// UserRepository interface defines how we interact with data sources
// The use case doesn't care about SQL vs NoSQL vs Memory
type UserRepository interface {
	Save(user User) error
	FindByID(id int) (*User, error)
}

// --- Use Case Layer (Application Logic) ---

type UserService struct {
	repo UserRepository
}

func NewUserService(r UserRepository) *UserService {
	return &UserService{repo: r}
}

func (s *UserService) RegisterUser(name string) error {
	// Business rule: Name cannot be empty
	if name == "" {
		return fmt.Errorf("name cannot be empty")
	}
	user := User{ID: 1, Name: name} // ID generation simplified
	return s.repo.Save(user)
}

// --- Infrastructure Layer (External Details) ---

type InMemoryUserRepo struct {
	store map[int]User
}

func (r *InMemoryUserRepo) Save(user User) error {
	if r.store == nil {
		r.store = make(map[int]User)
	}
	r.store[user.ID] = user
	fmt.Printf("Saved user %s to memory\n", user.Name)
	return nil
}

func (r *InMemoryUserRepo) FindByID(id int) (*User, error) {
	if user, ok := r.store[id]; ok {
		return &user, nil
	}
	return nil, fmt.Errorf("user not found")
}

// --- Main Wiring ---
func main() {
	// Dependency Injection
	repo := &InMemoryUserRepo{}
	service := NewUserService(repo)

	service.RegisterUser("Alice")
}
```

## Interview Questions
**Q: Explain the CAP Theorem.**
**A:** The CAP theorem states that a distributed data store can only provide two of the following three guarantees: Consistency (every read receives the most recent write), Availability (every request receives a response), and Partition Tolerance (system continues to operate despite network failures). In distributed systems, Partition Tolerance is usually non-negotiable, forcing a trade-off between Consistency and Availability (CP vs AP).

**Q: When would you choose a Monolith over Microservices?**
**A:** I would choose a Monolith for early-stage startups or simple applications where the domain is not well-defined, team size is small, and speed of initial development is critical. The complexity overhead of microservices (network latency, distributed tracing, deployment) is unjustified until the system requires independent scaling or distinct team ownership.

**Q: What is Vertical Scaling vs Horizontal Scaling?**
**A:** Vertical scaling (scaling up) involves adding more power (CPU, RAM) to an existing machine. Horizontal scaling (scaling out) involves adding more machines to a pool of resources. Horizontal scaling is generally preferred for distributed systems as it offers better fault tolerance and theoretical infinite scaling, though it introduces complexity.
