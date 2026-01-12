---
---

## Summary
The first rule of functions is that they should be small. The second rule of functions is that **they should be smaller than that**. Small methods and classes are easier to name, easier to understand, and easier to test. A massive file is a hiding place for bugs; a small file is a clear statement of intent.

## Detailed Explanation

### 1. Methods (Functions)
*   **Do One Thing**: A function should do one thing, do it well, and do it only.
*   **One Level of Abstraction**: Statements within a function should all be at the same level of abstraction. Don't mix high-level logic (`ProcessOrder`) with low-level details (`string.Split` or bitwise operations) in the same function.
*   **Size Heuristic**: Ideally 5-10 lines. If it doesn't fit on a screen without scrolling, it's too big.
*   **Arguments**: Ideally 0-2 arguments. 3 is a crowd. 4+ requires a struct/object.

### 2. Classes (Structs)
*   **Single Responsibility Principle (SRP)**: A class should have one reason to change.
*   **Cohesion**: Methods in a class should manipulate the class's variables. If a method doesn't use any of the class's variables, it probably belongs elsewhere (or should be static).
*   **Size Heuristic**: A class shouldn't implement an entire subsystem. If "User" class handles Auth, Database, and Email, it's a "God Class". Split it.

### 3. Files
*   **Navigation**: It is easier to find `OrderValidator.go` than to scroll to line 3000 of `OrderService.go`.
*   **Conflicts**: Large files cause more merge conflicts in version control.

## Go Code Examples

### The "Step-Down" Rule
Refactoring a large function into smaller, descriptive ones.

```go
// BAD: Mixed levels of abstraction, doing too much
func ProcessUser(u User) error {
    // Validation
    if len(u.Name) == 0 { return errors.New("empty name") }
    if u.Age < 18 { return errors.New("too young") }
    
    // DB Logic
    db, _ := sql.Open("postgres", "...")
    defer db.Close()
    _, err := db.Exec("INSERT INTO users ...", u.Name)
    
    // Email Logic
    msg := "Welcome " + u.Name
    smtp.SendMail("...", nil, "sender@example.com", []string{u.Email}, []byte(msg))
    
    return err
}

// GOOD: Small, one level of abstraction
func ProcessUser(u User) error {
    if err := validateUser(u); err != nil {
        return err
    }
    if err := saveUser(u); err != nil {
        return err
    }
    return sendWelcomeEmail(u)
}

// Helper functions (Low level details hidden)
func validateUser(u User) error { ... }
func saveUser(u User) error { ... }
func sendWelcomeEmail(u User) error { ... }
```

### Argument Objects
```go
// BAD: Too many arguments
func CreateMenu(title string, body string, buttonText string, cancellable bool) { ... }

// GOOD: Group concepts
type MenuConfig struct {
    Title       string
    Body        string
    ButtonText  string
    Cancellable bool
}
func CreateMenu(cfg MenuConfig) { ... }
```

## Interview Questions

**Q: How do you decide when to split a class?**
**A:** I look for **Cohesion**. If I have a class with 10 fields and 10 methods, but 5 methods only use the first 5 fields, and the other 5 methods use the other 5 fields, that's a clear sign that this is actually two classes stuck together. Also, if I can't describe the class without using the word "and" (e.g., "This class validates the user AND saves them"), it needs splitting.

**Q: Is there a performance cost to having many small functions?**
**A:** Technically, yes (stack frame overhead), but in 99% of cases, it is negligible compared to I/O or algorithmic complexity. The maintainability gain vastly outweighs the nanosecond cost. Modern compilers also inline small functions automatically.

**Q: What is the "Step-Down Rule"?**
**A:** Code should read from top to bottom like a narrative. High-level functions call lower-level functions. The definition of the lower-level function should appear immediately after the high-level function that calls it (or generally lower in the file), keeping the reader's flow consistent.
