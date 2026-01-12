---
---

## Summary
Frameworks are tools, not architectures. **Keep framework code distant** from your business logic. Use frameworks for what they are good at (routing, database connections, serialization), but do not let them invade your domain entities or rules. This is the core principle of **Clean Architecture**.

## Detailed Explanation

### 1. The Dependency Rule
Dependencies should point **inwards**. Your Core Domain (High Level) should not know about the Frameworks (Low Level) that run it.
*   **Wrong**: Your `User` entity imports `gorm.Model` or specific JSON tags.
*   **Right**: Your `User` entity is a plain struct. The database layer adapts it to SQL.

### 2. The Dangers of Coupling
If you inherit from framework classes or sprinkle framework annotations everywhere:
*   **Vendor Lock-in**: Switching from Gin to Fiber, or GORM to sqlx, requires rewriting the entire application.
*   **Testing Hell**: You can't unit test your logic without spinning up the entire framework container or database.
*   **Lifecycle**: Frameworks change. If your business logic depends on `Framework V1`, upgrading to `Framework V2` becomes a massive migration project.

### 3. Ports and Adapters (Hexagonal)
Treat the web (HTTP) just like any other delivery mechanism (CLI, gRPC). Your business logic shouldn't care if the request came from a Browser or a Cron job.

## Go Code Examples

### Framework Coupled (Bad)
Here, the Business Logic (Entity) is polluted with DB tags and Web tags.

```go
package domain

import "github.com/jinzhu/gorm"

// The Domain Entity is coupled to the DB (gorm) and the JSON presentation
type User struct {
    gorm.Model // Inheritance from Framework!
    Name string `json:"name" gorm:"column:user_name"`
    Token string `json:"token" gorm:"-"`
}

func (u *User) Save() {
    // Logic mixed with DB calls
    gorm.DB.Save(u) 
}
```

### Framework Decoupled (Good)
The Domain Entity is pure. Conversion happens at the boundaries.

```go
// --- DOMAIN PACKAGE (No imports!) ---
package domain

type User struct {
    ID   string
    Name string
}

type UserRepository interface {
    Save(u *User) error
}

// --- INFRASTRUCTURE PACKAGE ---
package postgres

// separate struct for DB mapping
type userModel struct {
    ID   string `gorm:"primary_key"`
    Name string `gorm:"column:user_name"`
}

// Adapter converts Domain -> DB Model
func toModel(u *domain.User) userModel {
    return userModel{ID: u.ID, Name: u.Name}
}
```

## Interview Questions

**Q: Why shouldn't I put SQL annotations (`gorm:"..."`) on my Domain Structs?**
**A:** Because it violates the Single Responsibility Principle. The Domain Struct should only change when business rules change. If you add tags, it now changes when the *Database Schema* changes. It tightly couples your business rules to your persistence layer.

**Q: Doesn't mapping between Domain objects and DTOs/Models create a lot of boilerplate code?**
**A:** Yes, it does. This is the trade-off. You pay the price of writing mappers (boilerplate) to gain the value of **Decoupling**. For small CRUD apps, it might be overkill (YAGNI). For long-lived enterprise systems, decoupling is vital for maintainability.

**Q: How do you handle HTTP Request/Response objects in Clean Architecture?**
**A:** They should never leave the "Delivery Layer" (Controllers/Handlers). The Controller should unmarshal the JSON into a agnostic Request Object (or simple arguments) and pass that to the Use Case. The Use Case should return a plain Result, which the Controller then marshals back to HTTP JSON.
