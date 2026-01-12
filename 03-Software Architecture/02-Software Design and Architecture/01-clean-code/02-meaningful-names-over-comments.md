---
---

## Summary
The most effective documentation is code that documents itself. **Meaningful names** allow a developer to read the code like prose, understanding the intent without jumping between definitions. "Comments are often a deodorant for smelly code." If you feel the need to write a comment to explain *what* a variable does, rename the variable instead.

## Detailed Explanation

### 1. Intention-Revealing Names
Names should answer: Why does it exist? What does it do? How is it used?
*   **Bad**: `var d int // elapsed time in days`
*   **Good**: `var elapsedTimeInDays int`

### 2. Avoid Disinformation
Do not refer to a grouping of accounts as an `accountList` unless it's actually a `List` data structure. If it's a map or a set, the name is misleading. Use `accounts` or `accountGroup`.

### 3. Make Meaningful Distinctions
Avoid noise words. `ProductInfo` or `ProductData` are not distinct from `Product`.
*   **Bad**: `getActiveAccount()`, `getActiveAccounts()`, `getActiveAccountInfo()` (Which one do I call?)
*   **Good**: `account`, `accounts`

### 4. Pronounceable Names
If you can't say it, you can't discuss it.
*   **Bad**: `genymdhms` (generation date, year, month, day, hour, minute, second)
*   **Good**: `generationTimestamp`

### 5. Comments: The Last Resort
Comments should explain **WHY**, not **WHAT** or **HOW**.
*   **Good Comment**: `// We use a predefined seed here to ensure tests are deterministic.`
*   **Bad Comment**: `// Increment i by 1` -> `i++`

## Go Code Examples

### Magic Numbers vs. Constants
```go
// BAD: What is 86400?
time.Sleep(86400 * time.Second)

// GOOD: Intent is clear
const SecondsInDay = 86400
time.Sleep(SecondsInDay * time.Second)
```

### Complex Conditionals vs. Variables
```go
// BAD: Hard to parse mentally
if employee.age > 65 && employee.yearsEmployed > 10 && !employee.isPartTime {
    // ...
}

// GOOD: Self-documenting
isEligibleForPension := employee.age > 65 && 
                        employee.yearsEmployed > 10 && 
                        !employee.isPartTime

if isEligibleForPension {
    // ...
}
```

### "Comment Deodorant"
```go
// BAD
// Check to see if the employee is eligible for full benefits
if (employee.flags & HOURLY_FLAG) && (employee.age > 65)

// GOOD
if employee.isEligibleForFullBenefits()
```

## Interview Questions

**Q: When is a comment absolutely necessary?**
**A:** 
1.  To explain **Why** a non-obvious decision was made (e.g., "Using this seemingly slow algorithm because it uses less memory, which is our constraint").
2.  To warn of consequences (e.g., "This function is not thread-safe").
3.  To mark `TODO` items.
4.  Documentation comments (Godoc) for public APIs.

**Q: What is the "Scope Rule" for variable names?**
**A:** The length of a variable name should be proportional to its scope. 
*   In a one-line loop, `i` is fine. 
*   In a global scope or large function, `i` is terrible; use `userIndex` or `retryCount`. 
*   Conversely, method names should be shorter if their class/package name provides context (`user.Name` vs `user.UserName`).

**Q: Why are "Hungarian Notation" or type prefixes (`strName`, `iCount`) discouraged in modern languages?**
**A:** Modern IDEs and statically typed languages (like Go, Java, TS) tell you the type instantly on hover or via compilation errors. Prefixes add noise and become lies when types change (e.g., changing `iCount` from int to long).
