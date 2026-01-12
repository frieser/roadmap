---
---

## Summary
**Interfaces** define a contract of behavior. They specify **what** an object can do, without specifying **how** it does it. They are the most powerful tool for decoupling systems in Object-Oriented Design.

## Detailed Explanation

### 1. The Contract
An interface is a promise. "If you implement `Reader`, you promise to have a `Read()` method."

### 2. Implicit vs. Explicit
*   **Explicit (Java/C#)**: `class File implements Reader`. You must declare intent.
*   **Implicit (Go)**: If `File` has a `Read()` method, it *is* a `Reader`. No declaration needed. This allows you to define interfaces for code you don't own (e.g., standard library types).

### 3. Interface Segregation
Keep interfaces small. "One method interfaces" are ideal.
*   `Reader` (Read)
*   `Writer` (Write)
*   `ReadWriter` (Read + Write) - Composition of interfaces.

## Go Application (Idiomatic Interfaces)

Go interfaces are structurally typed.

```go
// Defined by the Consumer (The code that needs it), not the Producer
type Stringer interface {
    String() string
}

type User struct {
    ID int
}

// Implicit implementation
func (u User) String() string {
    return fmt.Sprintf("User-%d", u.ID)
}

func PrintSomething(s Stringer) {
    fmt.Println(s.String())
}
```

### The "Accept Interfaces, Return Structs" Rule
In Go, functions should generally accept interfaces (to be flexible) but return concrete structs (to allow the caller to use all features).

## Interview Questions

**Q: What is the difference between an Interface and an Abstract Class?**
**A:**
*   **Interface**: Behavior only. No state (fields). Can implement multiple interfaces. Loose coupling.
*   **Abstract Class**: Behavior + State. Can define default implementations. Single inheritance. Tighter coupling.

**Q: What is a "Marker Interface"?**
**A:** An interface with no methods (e.g., `Serializable` in Java, or `interface{}`/`any` in Go). It is used to tag a class with metadata or to allow generic handling of any type. In Go, `any` allows a function to accept absolutely anything, but requires type assertions to use effectively.

**Q: Why are implicit interfaces (Go style) considered better for decoupling?**
**A:** Because you can define the interface *where it is used*, rather than where the type is defined. If I use a 3rd party library `LibA` that returns a `BigObject`, I can define a `SmallInterface` in my code that only includes the one method I need from `BigObject`. I don't need `LibA` to know about my interface.
