---
---

## Summary
**Polymorphism** (from Greek "many forms") is the ability of different objects to respond to the same message (method call) in their own way. It allows architects to design systems based on **contracts** (interfaces) rather than implementations, making the system extensible without modifying existing code (Open/Closed Principle).

## Detailed Explanation

### 1. Types of Polymorphism
*   **Ad-hoc (Overloading)**: Same function name, different arguments. (Not supported in Go).
*   **Parametric (Generics)**: Functions/Types written to handle values identically without depending on their type. (Supported in Go 1.18+).
*   **Subtype (Runtime/Dynamic)**: The most common OOP form. A variable of type `Shape` can hold a `Circle` or `Square`, and calling `.Draw()` executes the correct implementation at runtime.

### 2. Architectural Benefit
*   **Decoupling**: The High-Level Policy (e.g., `TaxCalculator`) depends on an interface (`TaxStrategy`), not the low-level details (`USATax`, `UKTax`).
*   **Testability**: Allows replacing real dependencies with Mocks or Stubs.

## Go Application (Interfaces)

Go achieves runtime polymorphism strictly through **Interfaces**. If a type implements all the methods of an interface, it implicitly implements that interface.

```go
package main

import "fmt"

// The Contract
type Notifier interface {
    Send(msg string) error
}

// Implementation 1: Email
type EmailNotifier struct {
    EmailAddr string
}

func (e *EmailNotifier) Send(msg string) error {
    fmt.Printf("Sending Email to %s: %s\n", e.EmailAddr, msg)
    return nil
}

// Implementation 2: SMS
type SMSNotifier struct {
    Phone string
}

func (s *SMSNotifier) Send(msg string) error {
    fmt.Printf("Sending SMS to %s: %s\n", s.Phone, msg)
    return nil
}

// Polymorphic Function
// Accepts ANY Notifier. Doesn't care if it's Email or SMS.
func AlertUser(n Notifier, msg string) {
    n.Send(msg)
}

func main() {
    email := &EmailNotifier{"user@example.com"}
    sms := &SMSNotifier{"555-0199"}
    
    // Both work
    AlertUser(email, "Server Down!")
    AlertUser(sms, "Server Down!")
}
```

## Interview Questions

**Q: Explain the difference between Compile-time and Runtime Polymorphism.**
**A:** 
*   **Compile-time (Static)**: Method Overloading. The compiler decides which method to call based on the argument types during compilation. (e.g., `Add(int, int)` vs `Add(string, string)`). Go does not support this.
*   **Runtime (Dynamic)**: Method Overriding/Interfaces. The specific implementation to be called is determined while the program is running (Dynamic Dispatch), typically using a vtable or itable.

**Q: How does Polymorphism enable the Open/Closed Principle?**
**A:** It allows you to extend the behavior of a system (add a new `Notifier`) without modifying the code that uses it (`AlertUser`). The `AlertUser` function is "Closed" for modification (you don't need to change it), but "Open" for extension (you can pass it new types of Notifiers).

**Q: What is "Duck Typing" and does Go use it?**
**A:** "If it walks like a duck and quacks like a duck, it's a duck." Dynamic languages (Python, Ruby) use this purely at runtime. Go uses **Structural Typing** for interfaces, which is a compile-time safe version of Duck Typing. You don't explicitly declare `implements Notifier`; if you define the method `Send`, you *are* a `Notifier`.
