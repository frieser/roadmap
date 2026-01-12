---
---

## Summary
The **Law of Demeter (LoD)**, or the **Principle of Least Knowledge**, states that a module should not know about the innards of the objects it manipulates. Succinctly: **"Only talk to your immediate friends."** It prevents tight coupling between disjoint parts of a system.

## Detailed Explanation

### 1. The Rule
A method `M` of object `O` should only invoke methods of:
1.  `O` itself.
2.  Parameters passed to `M`.
3.  Objects created within `M`.
4.  Direct component objects of `O`.

It should **NOT** invoke methods of objects returned by other calls.
*   **Violation**: `order.getCustomer().getAddress().getZipCode()` (Train Wreck).
*   **Compliance**: `order.getZipCode()` (Delegation).

### 2. Why it Matters
*   **Coupling**: In the violation above, the code depends on the structure of `Order`, `Customer`, AND `Address`. If `Customer` changes how it stores addresses, this code breaks.
*   **Maintenance**: Changes ripple through the system. "Train wrecks" make refactoring dangerous.

### 3. The Trade-off
Strict adherence can lead to **Wrapper/Proxy Methods** everywhere (e.g., `Order` needs `getZipCode`, `getCity`, `getState` just to delegate to `Customer.Address`).

## Go Application

### Violation (Train Wreck)
The caller knows too much about the internal structure of `Computer`.

```go
func GetDiskSize(c *Computer) int {
    // Violation: Accessing HardDrive via Processor via Computer
    return c.Processor.HardDrive.Size 
}
```

### Correction (Delegation)
The `Computer` struct should expose the capability, hiding the implementation details.

```go
type Computer struct {
    processor Processor
}

// Wrapper / Delegate method
func (c *Computer) DiskCapacity() int {
    return c.processor.hardDrive.Size
}

// Caller
func GetDiskSize(c *Computer) int {
    return c.DiskCapacity() // Only talks to Computer
}
```

## Interview Questions

**Q: Is fluent syntax (Builder Pattern) a violation of the Law of Demeter?**
**A:** Usually, no. The Law of Demeter applies to objects that return *other* objects (structure traversal). In a Fluent Interface (e.g., `Query.Where().OrderBy().Limit()`), each method returns `this` (the Query object itself). You are talking to the *same* friend repeatedly, which is allowed.

**Q: Does LoD apply to Data Structures (DTOs)?**
**A:** No. According to Clean Code (Robert Martin), LoD applies to **Objects** (behavior + data), not **Data Structures** (data only). If a struct is just a DTO with public fields, `a.b.c` is fine because there is no behavior to encapsulate or implementation to hide.
