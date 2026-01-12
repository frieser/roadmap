---
---

## Summary
The **Use Case Diagram** captures the **functional requirements** of a system. It describes *what* the system does from the user's perspective, not *how* it does it. It identifies the **Actors** (users or external systems) and their interactions with the system's **Use Cases** (goals/features).

## Detailed Explanation

Use Case diagrams are primarily used in the requirements gathering phase. They help stakeholders understand the scope of the system and ensure all user goals are met.

### Key Components

1.  **Actor**: An entity that interacts with the system.
    *   **Primary Actor**: Initiates the use case (e.g., User).
    *   **Secondary Actor**: Reacts or provides a service (e.g., External Payment Gateway).
    *   Notation: Stick figure.

2.  **Use Case**: A specific function or goal.
    *   Notation: Oval.

3.  **System Boundary**: Defines the scope of the system. Actors are outside, Use Cases are inside.
    *   Notation: Rectangle box around use cases.

4.  **Relationships**:
    *   **Association**: Solid line connecting Actor and Use Case.
    *   **Include**: Base use case *always* executes the included one (e.g., "Login" includes "Verify Password"). `<<include>>`
    *   **Extend**: Extending use case *optionally* executes (e.g., "Order" extends "Apply Coupon"). `<<extend>>`
    *   **Generalization**: Child actor/use case inherits behavior from parent.

### Usage
*   **Requirements Analysis**: Clarifying what needs to be built.
*   **Scope Definition**: Deciding what is inside vs. outside the system.
*   **Test Case Generation**: Use cases often map 1:1 to acceptance tests.

### Mermaid Example
```mermaid
useCaseDiagram
    actor User
    actor Admin
    package "E-Commerce System" {
        usecase "Login" as UC1
        usecase "View Products" as UC2
        usecase "Manage Users" as UC3
    }
    User --> UC1
    User --> UC2
    Admin --> UC1
    Admin --> UC3
```

## Go Example

Use Cases don't map directly to a specific code structure like Class Diagrams do, but in Go (and Clean Architecture), a Use Case is often implemented as a **Service** or **Handler** function.

```go
package main

import "fmt"

// Actor: User
type User struct {
	ID    string
	Email string
}

// Use Case: "Register User"
// In Go, this is often an interface defining the primary port/behavior
type UserRegistrationUseCase interface {
	Register(email, password string) error
}

// Implementation of the Use Case
type UserService struct {
	// dependencies like generic repository, email sender (Secondary Actors)
}

func (s *UserService) Register(email, password string) error {
	// 1. Validate Input (Include: Validate)
	if email == "" {
		return fmt.Errorf("invalid email")
	}

	// 2. Logic
	fmt.Printf("Registering user: %s\n", email)

	// 3. Notify (Association to Secondary Actor: Email System)
	// emailClient.Send(...)
	
	return nil
}

func main() {
	var uc UserRegistrationUseCase = &UserService{}
	uc.Register("jane@example.com", "secret")
}
```

## Interview Questions

### Q: What is the difference between `<<include>>` and `<<extend>>`?
**A:** `<<include>>` represents a mandatory relationship; the included use case is **always** executed as part of the base use case (e.g., "Rent Movie" includes "Check Availability"). `<<extend>>` is optional behavior that runs only under specific conditions (e.g., "Rent Movie" extends "Calculate Late Fee" only if late).

### Q: Can an Actor be a system?
**A:** Yes. An Actor is any entity external to the system boundary that interacts with it. This includes human users, other software systems (like a Bank API), or hardware devices.

### Q: Why are Use Case diagrams important if they don't show code?
**A:** They bridge the gap between technical developers and non-technical stakeholders (PMs, Clients). They ensure everyone agrees on the **features** and **scope** before any code is written, preventing scope creep.
