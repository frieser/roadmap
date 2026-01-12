---
---

## Summary
**Composition over Inheritance** is a design principle that suggests achieving polymorphic behavior and code reuse by combining objects with specific behaviors (**composition**) rather than inheriting from a base or parent class (**inheritance**). It promotes flexibility and loose coupling.

## Detailed Explanation

### The Case for Composition
In composition, a class (the "composite") holds a reference to one or more objects of other classes as its parts.
*   **Flexibility**: Behaviors can be swapped at runtime by changing the contained objects.
*   **Looser Coupling**: The composite class only depends on the interfaces of its parts, not their internal implementations.
*   **Avoids Hierarchical Rigidity**: You don't get stuck with a "God Object" parent that carries unnecessary baggage for all subclasses.

### The Problem with Inheritance (White-box Reuse)
*   **Tightly Coupled**: Subclasses are dependent on the internal implementation details of the parent.
*   **Fragile Base Class**: A change in the parent class can break subclasses in unexpected ways.
*   **Static**: Relationships are defined at compile-time and cannot change during execution.

### Go Implementation: Embedding
Go does not have inheritance. It uses **struct embedding** which is a form of syntactic sugar for composition.

```go
package main

import "fmt"

type Logger struct{}

func (l *Logger) Log(message string) {
	fmt.Println("LOG:", message)
}

type Database struct {
	Logger // Embedding: Database "Has-A" Logger
	Name   string
}

func main() {
	db := Database{Name: "Production"}
	// You can call Log directly on db because of embedding
	db.Log("Connecting to database " + db.Name)
}
```

## Interview Questions

### Q: Does Go support inheritance?
**A:** No, Go intentionally omits class-based inheritance to avoid its pitfalls. It uses **composition** (via embedding) and **interfaces** to achieve similar goals with more flexibility.

### Q: When should you still use inheritance (in languages that support it)?
**A:** When there is a clear, stable "Is-A" relationship and the hierarchy is shallow. However, even then, many architects recommend defaulting to composition.
