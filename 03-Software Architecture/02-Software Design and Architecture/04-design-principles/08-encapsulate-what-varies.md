---
---

## Summary
**Encapsulate What Varies** is a foundational design principle. It suggests identifying the parts of your application that are likely to change and separating them from the parts that will stay the same. This minimizes the impact of changes and promotes the **Open/Closed Principle**.

## Detailed Explanation

### 1. Identifying Variance
Changes usually come from:
*   **Business Rules**: Tax rates, discount logic.
*   **Infrastructure**: Database vendors, API providers.
*   **UI/Presentation**: Mobile vs. Web, JSON vs. XML.

### 2. Strategies
*   **Interface**: Hide the implementation behind an interface. (e.g., `Storage` interface hides `S3` vs `LocalDisk`).
*   **Strategy Pattern**: Encapsulate algorithms into separate classes.
*   **Configuration**: Move volatile values (URLs, timeouts) to config files.

### 3. Benefit
If you mix stable code (the core workflow) with volatile code (the specific API call), you have to modify and re-test the stable code every time the volatile part changes. Separating them protects the stable core.

## Go Application (Strategy Pattern)

Imagine a notification system. The *logic* of when to notify is stable. The *method* (Email, SMS, Slack) varies.

### Violation (Mixed Concerns)
```go
func NotifyUser(u User, msg string) {
    // Stable logic mixed with volatile implementation details
    if u.PreferEmail {
        // ... 20 lines of SMTP code ...
    } else if u.PreferSMS {
        // ... 20 lines of Twilio code ...
    }
}
```

### Correction (Encapsulated)
```go
// The "Concept" of notifying is stable. The implementation varies.
type Notifier interface {
    Send(u User, msg string) error
}

type EmailNotifier struct{} // Encapsulates SMTP logic
type SMSNotifier struct{}   // Encapsulates Twilio logic

// The core function doesn't change when we add Slack support later.
func NotifyUser(n Notifier, u User, msg string) {
    n.Send(u, msg)
}
```

## Interview Questions

**Q: How does this principle relate to Design Patterns?**
**A:** Almost all design patterns are implementations of this principle.
*   **Strategy**: Encapsulates varying algorithms.
*   **Factory**: Encapsulates object creation.
*   **Adapter**: Encapsulates incompatible interfaces.
*   **Observer**: Encapsulates the reaction to an event.

**Q: Should I encapsulate everything?**
**A:** No. "Encapsulate what *varies*." If something is unlikely to change (e.g., standard library string manipulation), wrapping it adds needless complexity. Prediction is hard, so often it's better to wait for the first change request before refactoring (See **YAGNI**).
