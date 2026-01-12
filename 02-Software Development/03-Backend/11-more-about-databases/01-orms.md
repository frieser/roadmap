---
---

## Summary
Object-Relational Mapping (ORM) is a technique that lets you query and manipulate data from a database using an object-oriented paradigm. In Go, ORMs map database tables to structs, enabling developers to interact with the database using Go code instead of writing raw SQL.

## Detailed Explanation
ORMs act as a bridge between the relational model of a database and the object model of a programming language.

### Pros of using an ORM
- **Productivity**: Reduces boilerplate code for CRUD operations.
- **Maintainability**: Makes it easier to handle schema changes.
- **Security**: Often provides built-in protection against SQL injection.
- **Abstraction**: Allows switching between different database engines (e.g., MySQL to Postgres) with minimal changes.

### Cons of using an ORM
- **Performance**: Can be slower than raw SQL for complex queries.
- **Complexity**: Hides the underlying SQL, making it harder to optimize or debug.
- **The "Impedance Mismatch"**: Objects and relational tables don't always map perfectly.

### Popular Go ORMs
- **GORM**: The most feature-rich and widely used ORM in the Go ecosystem. Supports associations, hooks, transactions, and more.
- **Ent**: An entity framework for Go developed by Facebook. It uses a graph-based schema and code generation to provide a type-safe API.
- **SQLBoiler**: A "database-first" ORM that generates code based on your existing database schema, ensuring type safety and performance.

## Go-specific Examples

### Basic CRUD with GORM
```go
package main

import (
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

type User struct {
    gorm.Model
    Name  string
    Email string `gorm:"uniqueIndex"`
}

func main() {
    dsn := "host=localhost user=gorm password=gorm dbname=gorm port=9920 sslmode=disable"
    db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{})

    // Create
    db.Create(&User{Name: "John Doe", Email: "john@example.com"})

    // Read
    var user User
    db.First(&user, 1) // find user with integer primary key 1

    // Update
    db.Model(&user).Update("Name", "John Smith")

    // Delete
    db.Delete(&user, 1)
}
```

### Type-safe Queries with Ent
Ent generates a schema that you can interact with like this:
```go
client.User.
    Query().
    Where(user.Name("John")).
    Only(ctx)
```

## Interview Questions
**Q: What is the main benefit of using a code-generation ORM like Ent over a reflection-based one like GORM?**
**A:** Code-generation ORMs provide better type safety and performance. Errors are caught at compile-time rather than runtime, and the generated code avoids the overhead of reflection during execution.

**Q: When would you choose raw SQL over an ORM in a Go project?**
**A:** Raw SQL is preferred for highly complex queries, performance-critical sections, or when you need to use database-specific features that the ORM doesn't support well. It gives you full control over the execution plan.

**Q: How do ORMs prevent SQL injection?**
**A:** Most ORMs use **parameterized queries** (prepared statements) by default. Instead of concatenating strings into a query, they send the query template and the user data separately to the database driver, ensuring data is never executed as code.
