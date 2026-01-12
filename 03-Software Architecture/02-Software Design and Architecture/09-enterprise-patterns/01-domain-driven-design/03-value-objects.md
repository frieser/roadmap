---
---

## Summary
A **Value Object** is an object that describes some characteristic or attribute but possesses no conceptual identity. It is defined strictly by its value. Two Value Objects are equal if all their fields are equal. They are typically **immutable**—operations on them return a new instance rather than modifying the existing one.

## Detailed Explanation

### Core Characteristics
1.  **No Identity**: There is no ID field. `Color(Red)` is always just Red.
2.  **Immutability**: Once created, it cannot change. This makes them thread-safe and side-effect free.
3.  **Self-Validating**: A Value Object should never exist in an invalid state. Validation happens at creation time.
4.  **Replaceability**: You don't update a value object; you replace it with a new one.

### Benefits
*   **Thread Safety**: Since they are immutable, they can be shared across threads without locking.
*   **expressiveness**: Using `Email` type instead of `string` clarifies intent and centralizes validation rules (avoids "Primitive Obsession").
*   **Bug Reduction**: Eliminates side effects caused by passing mutable references around.

## Go Example

```go
package domain

import "errors"

// Money is a Value Object.
type Money struct {
	amount   int64
	currency string
}

// NewMoney is the constructor that enforces validity.
func NewMoney(amount int64, currency string) (Money, error) {
	if currency == "" {
		return Money{}, errors.New("currency is required")
	}
	return Money{amount: amount, currency: currency}, nil
}

// Add ensures immutability by returning a NEW Money instance.
// It does NOT modify the receiver.
func (m Money) Add(other Money) (Money, error) {
	if m.currency != other.currency {
		return Money{}, errors.New("cannot add different currencies")
	}
	return Money{
		amount:   m.amount + other.amount,
		currency: m.currency,
	}, nil
}

// Equals checks value equality (Go structs compare by value automatically,
// but custom logic can be added if needed).
func (m Money) Equals(other Money) bool {
	return m == other
}
```

## Interview Questions

### Q: Why should Value Objects be immutable?
**A:** Immutability guarantees that the object's state is consistent and prevents side effects. If you pass a Value Object to another function, you don't have to worry about that function changing your local data. It also simplifies concurrency.

### Q: How do you persist Value Objects in a database?
**A:** They are typically embedded in the owning Entity's table. For example, a `User` table might have `address_street`, `address_city`, and `address_zip` columns to store the `Address` Value Object.
