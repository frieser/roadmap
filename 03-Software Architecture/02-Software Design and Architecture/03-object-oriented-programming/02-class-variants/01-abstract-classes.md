---
---

## Summary
**Abstract Classes** act as partial blueprints. They can define both implementation (methods with code) and interface (abstract methods without code). They cannot be instantiated directly. They are used to share common code among related classes while enforcing a specific contract.

## Detailed Explanation

### 1. Purpose
*   **Code Reuse**: Move common logic (e.g., `Log()`, `ID` field) to the abstract base class.
*   **Enforce Contract**: Force subclasses to implement specific methods (e.g., `CalculatePay()`).

### 2. Is-A Relationship
Abstract classes imply a strong "Is-A" relationship. `Dog` IS-A `Mammal`. Use them only when there is a clear hierarchy.

## Go Application (Simulation)

Go **does not** have `abstract` keywords. However, we can simulate this pattern using:
1.  **Interfaces** (The abstract part).
2.  **Struct Embedding** (The implementation sharing part).

### The Pattern: Interface + Base Struct

```go
package main

import "fmt"

// 1. The Abstract Contract
type Animal interface {
    Speak() string // Abstract method
    Move()         // Concrete method (provided by base)
}

// 2. The Base Implementation (Abstract Class equivalent)
type BaseAnimal struct {
    Name string
}

func (b *BaseAnimal) Move() {
    fmt.Println(b.Name, "is moving")
}

// 3. Concrete Implementation (Subclass)
type Dog struct {
    BaseAnimal // Inherit state and concrete methods
}

// Implement the "abstract" method
func (d *Dog) Speak() string {
    return "Woof"
}

func main() {
    d := Dog{BaseAnimal{Name: "Buddy"}}
    d.Move() // From Base
    fmt.Println(d.Speak()) // From Dog
}
```

## Interview Questions

**Q: When should you use an Abstract Class vs an Interface?**
**A:** 
*   Use an **Interface** when you want to define a capability (CanFly, CanSwim) across unrelated classes.
*   Use an **Abstract Class** (or Base Struct in Go) when you have strictly related classes (Car, Truck, Bus) that share a lot of implementation code (engine logic, wheel logic) but need to customize specific behaviors.

**Q: Can you instantiate an abstract class?**
**A:** No. It is incomplete by definition. In Go, you technically *can* instantiate the `BaseAnimal` struct, but it wouldn't satisfy the `Animal` interface because it lacks the `Speak()` method.
