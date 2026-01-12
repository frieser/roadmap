---
---

## Summary
**YAGNI (You Aren't Gonna Need It)** is a principle of Extreme Programming (XP) that states a programmer should not add functionality until deemed necessary. It warns against **speculative generality** and **premature optimization**, which bloat the codebase with unused features, increase complexity, and waste development time.

## Detailed Explanation

### 1. The Cost of Future-Proofing
Building a feature "just in case" incurs immediate costs:
*   **Development Time**: Time spent building unused features is time stolen from needed features.
*   **Maintenance**: Code needed to be tested, debugged, and migrated even if nobody uses it.
*   **Cognitive Load**: It makes the system harder to understand for new developers.

### 2. YAGNI vs. Extensibility
YAGNI does *not* mean writing rigid code. It means:
*   Make the code easy to change (Clean Code, SOLID).
*   Do *not* build the change itself until requested.
*   "Implement the simplest thing that could possibly work."

### 3. Architectural YAGNI
*   Don't build a Plugin System if you only have one hardcoded implementation.
*   Don't implement a complex Distributed Cache if a simple in-memory map suffices for current traffic.
*   Don't use Kubernetes for a simple CRUD app that runs on a single VPS.

## Go Application

### Violation (Speculative Generality)
Creating a generic interface for a Database when we only use Postgres and have no plans to switch.

```go
// BAD: Over-engineered "just in case" we switch DBs
type DBProvider interface {
    Connect(connString string) error
    Exec(query string, args ...interface{}) (Result, error)
    // ... 20 other methods
}

type MongoProvider struct{} // Implemented but never used
type OracleProvider struct{} // Implemented but never used
```

### Correction (Simple & Direct)
Use what you need. If you need to switch later, refactoring tools make it easy if the code is clean.

```go
// GOOD: Concrete implementation for the current reality
type PostgresDB struct {
    db *sql.DB
}

func NewPostgres(connStr string) (*PostgresDB, error) {
    // ...
}
```

## Interview Questions

**Q: How does YAGNI relate to TDD?**
**A:** TDD naturally enforces YAGNI. In TDD, you only write code to make a failing test pass. If there is no test requiring a feature (because there is no requirement for it), you don't write the code.

**Q: Is YAGNI an excuse for bad design?**
**A:** No. Skipping a feature because "YAGNI" is valid. Skipping good design practices (like separation of concerns) because "we might not need maintainability" is just laziness (Technical Debt). You always need maintainability.

**Q: When should you violate YAGNI?**
**A:** When the cost of retrofitting the feature later is astronomically higher than doing it now. For example, security and fundamental architectural choices (like choosing a language or basic data compliance) are hard to bolt on later.
