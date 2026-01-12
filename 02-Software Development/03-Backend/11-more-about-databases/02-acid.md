---
---

## Summary
ACID is a set of properties that guarantee database transactions are processed reliably. It stands for **Atomicity, Consistency, Isolation, and Durability**. Understanding ACID is crucial for building systems that require high data integrity, like banking or e-commerce platforms.

## Detailed Explanation

### 1. Atomicity ("All or Nothing")
A transaction is treated as a single unit. Either all its operations succeed, or none of them do. If any part fails, the entire transaction is rolled back to the previous state.

### 2. Consistency
A transaction must transition the database from one valid state to another, maintaining all predefined rules (constraints, cascades, triggers). It ensures that data remains accurate and meaningful according to the business logic.

### 3. Isolation
Transactions occur independently of each other. The intermediate state of a transaction is invisible to other concurrently running transactions. This prevents issues like "dirty reads."

### 4. Durability
Once a transaction is committed, it remains committed even in the event of a system failure (e.g., power loss). Changes are permanently recorded in non-volatile memory (disk).

### ACID in Relational Databases
Most RDBMS like PostgreSQL, MySQL (with InnoDB), and SQL Server are fully ACID-compliant. NoSQL databases often relax some of these properties (following the BASE model) to achieve higher scalability.

## Go-specific Context
In Go, ACID properties are managed through the `sql.Tx` object or through an ORM's transaction API.

### Ensuring Atomicity in Go
```go
tx, err := db.Begin()
if err != nil {
    return err
}

// All operations inside tx must succeed
_, err = tx.Exec("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
if err != nil {
    tx.Rollback() // Rollback ensures Atomicity
    return err
}

_, err = tx.Exec("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
if err != nil {
    tx.Rollback()
    return err
}

return tx.Commit() // Commit makes changes permanent (Durability)
```

## Interview Questions
**Q: Explain the 'Isolation' level and name a few common ones.**
**A:** Isolation levels define how transaction integrity is visible to other users/systems. Common levels include: **Read Uncommitted**, **Read Committed** (default in Postgres), **Repeatable Read**, and **Serializable** (strongest).

**Q: What is a 'Dirty Read'?**
**A:** A dirty read occurs when a transaction reads data that has been modified by another transaction but not yet committed. If the other transaction is rolled back, the first transaction has read invalid data.

**Q: Does every database need to be ACID-compliant?**
**A:** No. Some distributed databases prioritize availability and performance over strict consistency (the CAP theorem). These systems often use **Eventual Consistency** (part of the BASE model), which is acceptable for use cases like social media feeds or analytics.
