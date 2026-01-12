---
---

## Summary
**Inheritance** is an OOP mechanism where a new class (Subclass/Child) derives properties and behaviors from an existing class (Superclass/Parent). While it enables code reuse and polymorphism, excessive use leads to rigid, fragile architectures. Modern design strongly favors **Composition over Inheritance**.

## Detailed Explanation

### 1. "Is-A" Relationship
Inheritance models an "Is-A" relationship.
*   `Dog` is an `Animal`.
*   `Manager` is an `Employee`.
If the statement "B is an A" doesn't hold true in all contexts (Liskov Substitution Principle), inheritance is the wrong choice.

### 2. The Problems with Inheritance
*   **The Diamond Problem**: (In languages with multiple inheritance like C++) Ambiguity arises when two parent classes implement the same method.
*   **Fragile Base Class**: Changes to the parent class often break the child classes unexpectedly, creating tight coupling.
*   **Hierarchy Abuse**: Creating deep taxonomies (e.g., `AbstractEnterpriseControllerFactory`) just to share code makes the system hard to navigate.

### 3. Inheritance in Architecture
Architects use inheritance primarily for:
*   **Polymorphism**: Defining a common interface or abstract base class so clients can treat different objects uniformly.
*   **Frameworks**: Many frameworks (like Java Swing or Django) require you to inherit from base classes to hook into their lifecycle.

## Go Application (Embedding)

Go **does not** support classical inheritance (no `extends`). Instead, it uses **Struct Embedding** (Composition) to achieve similar goals (code reuse and interface satisfaction).

### Composition masquerading as Inheritance
```go
type Animal struct {
    Name string
}

func (a *Animal) Move() {
    fmt.Println(a.Name, "is moving")
}

// Dog "embeds" Animal. It gains Animal's fields and methods.
type Dog struct {
    Animal // Anonymous field (Embedding)
    Breed  string
}

func main() {
    d := Dog{
        Animal: Animal{Name: "Buddy"},
        Breed:  "Pug",
    }
    
    // Looks like inheritance: we call Move() directly on Dog
    d.Move() 
    
    // But it's actually composition:
    // d.Animal.Move() is what's happening under the hood.
}
```

### Key Difference
In Go, `Dog` is **NOT** an `Animal`. You cannot pass a `*Dog` to a function expecting `*Animal`. Embedding provides **syntactic sugar** for delegation, not subtyping. To achieve subtyping, you must use **Interfaces**.

## Interview Questions

**Q: Why does the phrase "Favor Composition over Inheritance" exist?**
**A:** Because inheritance creates the tightest coupling possible in OO design. You cannot change the parent without affecting the child. Composition ("Has-A") allows you to swap behavior at runtime (Dependency Injection), makes testing easier (Mocking), and keeps classes focused on a single responsibility.

**Q: Does Go support Multiple Inheritance?**
**A:** Not in the classical sense. However, a struct can embed multiple other structs. If both embedded structs have a method with the same name, the compiler will complain only if you try to call that ambiguous method, forcing you to resolve it explicitly (e.g., `d.Animal.Move()` vs `d.Robot.Move()`).

**Q: What is the Liskov Substitution Principle (LSP) in the context of inheritance?**
**A:** It states that objects of a superclass should be replaceable with objects of its subclasses without breaking the application. If a `Square` inherits from `Rectangle` but changing its width also changes its height (violating the Rectangle's behavior), it violates LSP. Inheritance should only be used when the subclass truly adheres to the parent's contract.
