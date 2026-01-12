---
---

## Summary
**Concrete Classes** are the workhorses of OOP. They are fully implemented types that can be instantiated. They provide the complete definition for objects, including all attributes and methods.

## Detailed Explanation

### 1. Characteristics
*   **Instantiable**: You can create an object of this type (`new User()`).
*   **Complete**: No abstract methods. All behavior is defined.
*   **Final (Optional)**: In some designs, concrete classes are marked `final` (cannot be subclassed) to favor composition.

### 2. Role in Architecture
Concrete classes should exist at the **edges** or **lowest levels** of your architecture. High-level policies should depend on Interfaces, not Concrete Classes (Dependency Inversion). If your business logic depends directly on `PostgresDatabase` (Concrete), you are locked in.

## Go Application (Structs)

In Go, almost all structs are "Concrete Classes" by default.

```go
type User struct {
    Name string
    Email string
}

func (u User) String() string {
    return fmt.Sprintf("%s <%s>", u.Name, u.Email)
}

// Usage
u := User{Name: "Alice", Email: "alice@example.com"} // Instantiation
```

### Avoiding "Concrete Dependency"
```go
// BAD: Depending on concrete type
func SendWelcome(u User, m GmailSender) { ... }

// GOOD: Depending on interface (satisfied by concrete type)
func SendWelcome(u User, m EmailSender) { ... }
```

## Interview Questions

**Q: What is the "Fragile Base Class" problem?**
**A:** It occurs when a concrete class is used as a base class for inheritance. If the base class changes its internal behavior, it can inadvertently break the subclasses that depend on that behavior, even if the public API didn't change. This is why "Composition > Inheritance" and "Design for inheritance or prohibit it (final)" are common mantras.

**Q: Should you test Concrete Classes or Interfaces?**
**A:** You write tests *against* the public API of Concrete Classes to verify they fulfill the contract. However, when writing tests for *other* classes that depend on this one, you should mock the Interface, not the Concrete Class.
