---
---

## Summary
**Encapsulation** is the bundling of data (attributes) and methods (behavior) that operate on that data into a single unit, and restricting direct access to some of that object's components. It protects the integrity of the data (Invariants) and allows the internal representation to change without affecting external clients.

## Detailed Explanation

### 1. The Guard Dog
Encapsulation acts as a guard. It ensures that an object is never in an invalid state.
*   *Without Encapsulation*: `user.Age = -100` (The system is now broken).
*   *With Encapsulation*: `user.SetAge(-100)` -> returns Error "Age cannot be negative".

### 2. Separation of Interface and Implementation
*   **Public Interface**: The set of methods exposed to the world.
*   **Private Implementation**: The internal fields and helper logic.
You can optimize the internal storage (e.g., changing a `List` to a `Map` for performance) without breaking any code that calls the public methods.

### 3. Architectural Benefit
Encapsulation reduces **Coupling**. If external code reaches directly into your object's fields, it is tightly coupled to your data structure. If it goes through methods, it is only coupled to your method signature.

## Go Application (Unexported Fields)

Go uses package-level visibility (Capitalized vs. Lowercase) to enforce encapsulation.

```go
package bank

// Account encapsulates the balance.
// 'balance' is unexported (private), so external packages CANNOT modify it directly.
type Account struct {
    balance int64 
    ID      string // Exported (public), safe to read/write? Maybe not, but it is allowed.
}

// Constructor ensures object starts in valid state
func NewAccount(startBalance int64) (*Account, error) {
    if startBalance < 0 {
        return nil, fmt.Errorf("negative balance")
    }
    return &Account{balance: startBalance}, nil
}

// Method protects the invariant (balance cannot be negative)
func (a *Account) Withdraw(amount int64) error {
    if amount > a.balance {
        return fmt.Errorf("insufficient funds")
    }
    a.balance -= amount
    return nil
}

// Getter (Read-only access)
func (a *Account) Balance() int64 {
    return a.balance
}
```

## Interview Questions

**Q: Is Encapsulation just about hiding data?**
**A:** No, that's "Information Hiding." Encapsulation is also about *bundling* behavior with data. A struct with public fields and no methods has no encapsulation (it's just a data structure). A struct with private fields and methods that enforce business rules has encapsulation.

**Q: Why are "Getters and Setters" for every field considered an anti-pattern (Anemic Model)?**
**A:** If you simply generate `GetX()` and `SetX()` for every private field `x`, you have technically achieved "private fields," but you haven't achieved encapsulation. You are still exposing the implementation details. You should expose *behavior* (`PromoteUser()`), not data (`SetRank()`).

**Q: How does Go's encapsulation differ from Java/C++?**
**A:** Go's boundary is the **Package**, not the Class. Code in `file_a.go` can access private fields of a struct defined in `file_b.go` if they are in the same package. This allows for "friend" relationships between closely related types without special keywords.
