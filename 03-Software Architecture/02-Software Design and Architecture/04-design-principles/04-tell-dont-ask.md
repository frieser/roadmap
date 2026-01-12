---
---

## Summary
**Tell, Don't Ask** is a principle that helps achieve **Encapsulation**. It suggests that instead of asking an object for data and acting on it (`if obj.Data == x`), you should tell the object what to do (`obj.DoAction()`). This keeps behavior close to the data it operates on (Cohesion).

## Detailed Explanation

### 1. The Problem: Anemic Models
"Asking" usually leads to **Anemic Domain Models**—objects that are bags of getters/setters with no logic. The logic is scattered across "Service" or "Manager" classes, violating the Information Hiding principle.

### 2. The Solution: Rich Models
"Telling" moves the logic *into* the object. The caller invokes a high-level command, and the object decides how to execute it based on its internal state.

### 3. Exceptions
*   **DTOs (Data Transfer Objects)**: These are purely for data transport. "Asking" for their data is their entire purpose.
*   **Query Methods**: Sometimes you genuinely need to display data (UI) or report on state.

## Go Application

### Violation (Ask)
The caller pulls data out, processes it, and puts it back.

```go
// BAD: The logic is outside the struct
func ChargeCustomer(w *Wallet, amount int) error {
    if w.Balance < amount { // Asking for state
        return errors.New("insufficient funds")
    }
    w.Balance -= amount // Modifying state directly
    return nil
}
```

### Correction (Tell)
The caller tells the struct what to do. The struct protects its own invariants.

```go
// GOOD: Logic is encapsulated
func (w *Wallet) Debit(amount int) error {
    if w.balance < amount {
        return errors.New("insufficient funds")
    }
    w.balance -= amount
    return nil
}

// Caller code
err := wallet.Debit(100)
```

## Interview Questions

**Q: How does "Tell, Don't Ask" relate to the Law of Demeter?**
**A:** They are closely related. Law of Demeter says "don't talk to strangers" (don't chain getters like `a.getB().getC().do()`). "Tell, Don't Ask" fixes this by moving the `do()` method up the chain. Instead of digging into `B` and `C`, you tell `A` to `do()`, and `A` delegates to `B`, which delegates to `C`.

**Q: Can "Tell, Don't Ask" lead to large classes?**
**A:** Yes, if taken to the extreme, objects can become "God Objects" that handle every possible operation on their data. It's a balance. Use **Composition** or **Visitor Pattern** to extract complex behaviors if the class grows too large, while still keeping the data encapsulated.
