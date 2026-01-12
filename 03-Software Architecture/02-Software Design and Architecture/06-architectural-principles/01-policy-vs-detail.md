---
---

## Summary
The separation of **Policy** from **Detail** is the central theme of clean architecture. **Policy** is the high-level business logic (the "What" and "Why"). **Detail** is the low-level implementation (the "How"). Good architecture ensures that Policy never depends on Detail; instead, Detail should plug into Policy.

## Detailed Explanation

### 1. What is Policy?
Policy embodies the core business rules and use cases. It represents the value of the software.
*   *Example*: "If a user's credit score is > 700, approve the loan."
*   *Characteristics*: Stable, rarely changes unless business goals change, language-agnostic.

### 2. What is Detail?
Detail is the mechanism used to execute the policy. It is necessary but secondary.
*   *Example*: "Store the loan application in PostgreSQL," "Serve the approval via a REST API," "Display the result in React."
*   *Characteristics*: Volatile, changes frequently (UI refreshes, DB migrations), technology-specific.

### 3. The Dependency Rule
In a well-architected system, source code dependencies must point only **inward**, toward higher-level policies.
*   **Wrong**: `LoanApprover` (Policy) imports `PostgresDriver` (Detail).
*   **Right**: `LoanApprover` defines `LoanRepository` interface. `PostgresLoanRepository` (Detail) implements it.

### 4. Delayed Decisions
A good architecture allows you to delay decisions about details. You should be able to write and test your entire Policy (Business Logic) without deciding whether you'll use Mongo, SQL, or a flat file.

## Go Application (Hexagonal Style)

```go
// --- POLICY (Core Domain) ---
// Note: No imports from "drivers" or "frameworks"

type LoanApplication struct {
    CreditScore int
    Amount      int
}

// The Policy depends on an abstraction (Interface), not a detail.
type NotificationService interface {
    Notify(msg string) error
}

func ApproveLoan(app LoanApplication, notifier NotificationService) bool {
    if app.CreditScore > 700 {
        notifier.Notify("Approved!") // Policy Logic
        return true
    }
    return false
}

// --- DETAIL (Infrastructure) ---
// Note: Imports Policy

type SMSNotifier struct {
    TwilioKey string
}

func (s *SMSNotifier) Notify(msg string) error {
    // Detail Logic (Twilio API calls)
    fmt.Println("Sending SMS via Twilio:", msg)
    return nil
}

func main() {
    // Wiring Policy and Detail together
    app := LoanApplication{CreditScore: 750}
    detail := &SMSNotifier{TwilioKey: "123"}
    
    ApproveLoan(app, detail)
}
```

## Interview Questions

**Q: Why is the Database considered a "Detail"? Isn't the data the most important part?**
**A:** The *Data* is important (Policy), but the *Database* (the software: Oracle, MySQL) is a detail. The data model is a high-level concept. The mechanism of storage (B-Trees, WAL logs, SQL dialect) is a low-level detail. Your business rules shouldn't break just because you switched from MySQL 5.7 to 8.0 or migrated to DynamoDB.

**Q: How does this principle relate to Frameworks (e.g., Gin, Spring)?**
**A:** Frameworks are details. They are delivery mechanisms. Your business objects should not inherit from framework classes or be annotated with framework-specific tags. If you marry your policy to a framework, you die with the framework.

**Q: What is the benefit of "Delayed Decisions"?**
**A:** It reduces risk. In the early stages of a project, you have the least information. Committing to a specific database or message broker early on is a guess. By keeping the Policy decoupled, you can use a simple in-memory implementation for months while you learn the requirements, then choose the perfect persistent storage later when you have real data on the load profile.
