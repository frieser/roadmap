---
---

## Summary
**Scope and Visibility** (Access Modifiers) control which parts of a program can access specific data or methods. This is the primary mechanism for **Encapsulation**. By hiding internal state and exposing only a defined API, architects prevent tight coupling and ensure that objects maintain their own invariants.

## Detailed Explanation

### 1. Common Visibility Levels (General OOP)
*   **Public**: Accessible from anywhere. This is the "API" of the object.
*   **Private**: Accessible only from within the class itself. Used for internal state and helper methods.
*   **Protected**: Accessible from within the class and its subclasses (Inheritance).
*   **Package/Internal**: Accessible from within the same module or package.

### 2. The Architectural Rule: "Least Privilege"
Always choose the most restrictive visibility possible. Make everything `private` by default. Only promote to `protected` or `public` if absolutely necessary.
*   **Why?** Public members represent a commitment. Once something is public, other code relies on it, and you can't change it easily. Private members can be refactored or deleted at will without breaking external clients.

### 3. Encapsulation vs. Information Hiding
*   **Encapsulation**: Bundling data and methods together.
*   **Information Hiding**: Using visibility to prevent access to that data.
Scope ensures that the implementation details (How it works) are hidden from the consumer (What it does).

## Go Application (Exported vs. Unexported)

Go simplifies the complex matrix of `public/private/protected` into a single rule based on **Capitalization**.

*   **Uppercased** (e.g., `User`): **Exported** (Public). Accessible to any package.
*   **Lowercased** (e.g., `user`): **Unexported** (Package-Private). Accessible only within the same package.

Go **does not** have `protected` or `private` (class-level). All unexported identifiers are visible to the entire package.

```go
package domain

// Exported (Public)
type Account struct {
    Owner string  // Public field
    id    string  // Private field (Package-Private)
    balance int64 // Private field
}

// Exported Method (Public API)
func (a *Account) Deposit(amount int64) {
    a.logTransaction(amount) // Internal call
    a.balance += amount
}

// Unexported Method (Private helper)
// Only visible inside package 'domain'
func (a *Account) logTransaction(amount int64) {
    // ...
}
```

### Architectural Implication in Go
Since Go's visibility is at the **Package** level, not the **Type** level, you can have "friend" classes implicitly if they are in the same package.
*   **Design Tip**: Keep packages small and focused. If a package has too many files accessing each other's private internals, it's a sign of low cohesion.

## Interview Questions

**Q: Why doesn't Go have a "private" keyword?**
**A:** Go designers aimed for simplicity and readability. The capitalization rule means you can tell if a symbol is public just by looking at its name, without needing to jump to its definition or use an IDE. It also enforces a strict separation between the public API (godoc) and internal implementation.

**Q: What is the risk of making fields public?**
**A:** It breaks Encapsulation. If a field is public, external code can set it to an invalid state (e.g., setting `age = -5`). It also prevents you from adding logic later (like validation or notifying listeners) when the value changes, because you didn't use a setter method.

**Q: How do you simulate "Protected" visibility in Go?**
**A:** You can't strictly enforce it. However, you can define an interface with an unexported method. Only types within the defining package can implement that interface, effectively restricting who can satisfy the interface "contract," which is a form of access control.
