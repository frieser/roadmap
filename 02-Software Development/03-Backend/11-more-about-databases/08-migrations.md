---
---

## Summary
Database Migrations are version control for your database schema. They allow you to define, evolve, and share your database structure in a reproducible way across different environments (development, staging, production).

## Detailed Explanation
Instead of manually running SQL commands on the server, migrations are stored as files that describe changes (up) and how to revert them (down).

### Why use Migrations?
-   **Consistency**: Ensures all team members and environments have the same schema.
-   **Automation**: Allows the schema to be updated automatically during CI/CD.
-   **History**: Tracks who changed what and when.

### Common Go Migration Tools
-   **golang-migrate**: The most popular tool. Uses raw SQL files and a simple versioning system. Supports many database drivers.
-   **goose**: Flexible tool that supports both SQL migrations and migrations written in Go code.
-   **GORM AutoMigrate**: Automatically syncs structs to tables. Good for rapid prototyping, but risky for production as it won't handle renames or complex changes.

## Go-specific Examples

### SQL Migration Example (golang-migrate)
`000001_create_users_table.up.sql`:
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

`000001_create_users_table.down.sql`:
```sql
DROP TABLE users;
```

### Running Migrations in Go code
```go
import (
    "github.com/golang-migrate/migrate/v4"
    _ "github.com/golang-migrate/migrate/v4/database/postgres"
    _ "github.com/golang-migrate/migrate/v4/source/file"
)

func main() {
    m, _ := migrate.New(
        "file://migrations",
        "postgres://user:pass@localhost:5432/db?sslmode=disable",
    )
    if err := m.Up(); err != nil && err != migrate.ErrNoChange {
        log.Fatal(err)
    }
}
```

## Interview Questions
**Q: Should you use GORM's `AutoMigrate` in production?**
**A:** Generally, no. `AutoMigrate` is non-destructive (it won't delete columns) and can be unpredictable. For production, it's safer to use explicit migration files where every change is reviewed and tested.

**Q: What are 'Down' migrations and are they always necessary?**
**A:** Down migrations are the reverse of Up migrations. They allow you to "roll back" the database to a previous state. While highly recommended, some complex migrations (like those involving data transformation) can be very difficult to revert reliably.

**Q: How do you handle a failed migration in production?**
**A:** A robust migration tool should wrap the migration in a transaction. If it fails, the database remains in the state it was before the migration started. If it doesn't support transactions, you must manually repair the state based on the migration logs.
