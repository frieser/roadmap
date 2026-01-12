---
---

## Summary
A Database Transaction is a sequence of one or more SQL operations executed as a single logical unit of work. Transactions are the mechanism used to implement the ACID properties, ensuring that complex data modifications are handled safely and consistently.

## Detailed Explanation
Transactions are essential when multiple database operations must happen together to maintain data integrity. The classic example is a bank transfer: subtracting money from one account and adding it to another must happen as one atomic step.

### Transaction Lifecycle
1.  **BEGIN**: Marks the start of the transaction.
2.  **EXECUTE**: Run various SQL statements (Insert, Update, Delete).
3.  **COMMIT**: Saves the changes permanently if everything succeeded.
4.  **ROLLBACK**: Aborts the transaction and reverts all changes if an error occurred.

### Isolation Levels and Concurrency
Databases allow configuring how isolated transactions are from each other to balance performance and safety:
-   **Read Committed**: Prevents dirty reads.
-   **Repeatable Read**: Prevents non-repeatable reads.
-   **Serializable**: Prevents phantom reads (highest safety, lowest concurrency).

## Go-specific Examples

### Standard Library Transaction
Using the `database/sql` package:
```go
func transferFunds(db *sql.DB, fromID, toID int, amount float64) error {
    tx, err := db.BeginTx(context.Background(), &sql.TxOptions{
        Isolation: sql.LevelSerializable,
    })
    if err != nil {
        return err
    }
    // Defer rollback in case of panic or early return
    defer tx.Rollback()

    if _, err := tx.Exec("UPDATE accounts SET balance = balance - $1 WHERE id = $2", amount, fromID); err != nil {
        return err
    }

    if _, err := tx.Exec("UPDATE accounts SET balance = balance + $1 WHERE id = $2", amount, toID); err != nil {
        return err
    }

    return tx.Commit()
}
```

### Transaction with GORM
GORM provides a simpler way to wrap operations in a transaction:
```go
err := db.Transaction(func(tx *gorm.DB) error {
    if err := tx.Create(&User{Name: "Alice"}).Error; err != nil {
        return err // Returning error will rollback automatically
    }
    if err := tx.Create(&User{Name: "Bob"}).Error; err != nil {
        return err
    }
    return nil // Returning nil will commit
})
```

## Interview Questions
**Q: Why should you use `defer tx.Rollback()` in Go?**
**A:** Calling `tx.Rollback()` is harmless if `tx.Commit()` has already been called. However, if the function returns early due to an error or panics before committing, the deferred rollback ensures that the transaction is closed and resources are released.

**Q: What is a 'Deadlock' in the context of transactions?**
**A:** A deadlock occurs when two transactions are waiting for each other to release locks on resources. For example, Tx1 locks Row A and wants Row B, while Tx2 locks Row B and wants Row A. Databases usually detect this and abort one of the transactions.

**Q: What are 'Nested Transactions'?**
**A:** Nested transactions allow a transaction to contain "sub-transactions." In Go, GORM supports this using **Savepoints**. If a sub-transaction fails, you can roll back to a specific savepoint without aborting the entire parent transaction.
