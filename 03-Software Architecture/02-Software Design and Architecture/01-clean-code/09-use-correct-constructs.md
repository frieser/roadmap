---
---

## Summary
**Use Correct Constructs** means choosing the right tool, data structure, or language feature for the job. Using a `List` when you need a `Set` (uniqueness), using `float` for currency (precision loss), or using `Exceptions` for control flow are all examples of using the wrong constructs. This principle improves performance, correctness, and readability.

## Detailed Explanation

### 1. Data Structures Matter
*   **Map (Dictionary)**: Use for O(1) lookups. Don't iterate over a list to find an ID.
*   **Set**: Use for uniqueness and membership checks.
*   **Stack/Queue**: Use for LIFO/FIFO logic (e.g., backtracking or job processing).

### 2. Type Safety
*   **Enums/Constants**: Don't use "Magic Strings" (`"admin"`, `"user"`). Use typed constants to prevent typos.
*   **Value Objects**: Don't use generic `string` for everything. A `Email` type is safer than a `string` that happens to hold an email.

### 3. Floating Point Math
*   **Money**: NEVER use `float` or `double` for money. 0.1 + 0.2 != 0.3 in IEEE 754. Use `int` (cents) or a `Decimal` type.

## Go Code Examples

### Map vs. Slice for Lookup
```go
// BAD: O(N) Lookup
func hasID(ids []string, target string) bool {
    for _, id := range ids {
        if id == target {
            return true
        }
    }
    return false
}

// GOOD: O(1) Lookup
func hasID(ids map[string]bool, target string) bool {
    _, exists := ids[target]
    return exists
}
```

### Money: Float vs. Int
```go
// BAD: Precision errors
price := 19.99
quantity := 3
total := price * float64(quantity) // might be 59.97000000000001

// GOOD: Use Integers (Cents)
priceCents := 1999
quantity := 3
totalCents := priceCents * quantity // 5997
fmt.Printf("Total: $%.2f", float64(totalCents)/100.0)
```

### Idiomatic Sets in Go
Go doesn't have a built-in `Set` type, so we use `map[T]struct{}`.
*   `struct{}` takes 0 bytes of memory.
*   `bool` takes 1 byte.
*   Therefore, `map[string]struct{}` is the correct construct for a Set, not `map[string]bool`.

```go
set := make(map[string]struct{})
set["item1"] = struct{}{}

if _, ok := set["item1"]; ok {
    fmt.Println("Exists")
}
```

## Interview Questions

**Q: Why shouldn't you use `float` for financial calculations?**
**A:** Floating point numbers (IEEE 754) trade precision for range. They cannot accurately represent some decimal fractions (like 0.1) in binary. Repeated operations accumulate these small errors, leading to missing cents in financial reports. Always use Integers (smallest unit) or Arbitrary-precision decimal libraries.

**Q: When should you use a Linked List over an Array/Slice?**
**A:** Almost never in modern systems, due to CPU cache locality. Arrays/Slices are contiguous in memory, making them extremely fast to traverse. Linked Lists cause cache misses because nodes are scattered in memory. You only use Linked Lists if you need O(1) insertions/deletions *in the middle* of the list and have iterators already at that position.

**Q: What is the "Semantics" of a data structure?**
**A:** It's the intent communicated by the choice of structure. Using a `Set` tells the reader "Order doesn't matter, and duplicates are impossible." Using a `List` tells the reader "Order matters, and duplicates might exist." Choosing the correct construct documents the data's nature.
