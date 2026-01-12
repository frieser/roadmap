---
---

## Summary
Database Failure Modes refers to the different ways a database system can fail and how an application should respond to maintain availability and data integrity. Understanding these modes is essential for building resilient backend systems.

## Detailed Explanation

### Common Failure Modes
1.  **Connection Failures**: Network issues or the database reaching its maximum connection limit.
2.  **Resource Exhaustion**: The database server runs out of CPU, RAM, or Disk space.
3.  **Slow Queries**: A query locks a table or consumes too many resources, causing other requests to timeout.
4.  **Hardware Failure**: Disk crash, power failure, or node failure in a distributed system.
5.  **Software Bugs**: Corruption in the database engine or misconfiguration.

### Resilience Patterns
-   **Retry Logic**: Automatically retrying a failed operation with exponential backoff.
-   **Circuit Breaker**: Stopping requests to the database if it is consistently failing, allowing it time to recover.
-   **Timeouts**: Ensuring the application doesn't wait indefinitely for a response.
-   **Connection Pooling**: Managing and reusing database connections efficiently.

## Go-specific Context
The `database/sql` package and Go's concurrency model provide tools to handle these failures.

### Handling Timeouts with Context
Always pass a `context.Context` with a timeout to database calls.
```go
ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

rows, err := db.QueryContext(ctx, "SELECT * FROM large_table")
if err != nil {
    if errors.Is(err, context.DeadlineExceeded) {
        // Handle timeout
    }
}
```

### Connection Pool Configuration
```go
db.SetMaxOpenConns(25)   // Limit total connections to avoid resource exhaustion
db.SetMaxIdleConns(25)   // Keep some connections open for performance
db.SetConnMaxLifetime(5 * time.Minute)
```

## Interview Questions
**Q: What is a 'Poison Pill' query?**
**A:** A query that is so complex or poorly optimized that it causes the database to crash or hang whenever it is executed, effectively taking down the service.

**Q: How does a Circuit Breaker help in a database failure scenario?**
**A:** If the database starts failing, the circuit breaker "opens." Instead of sending more requests and overwhelming the struggling DB, the application returns an immediate error (or cached data). Once the DB is healthy again, the circuit "closes" and resumes traffic.

**Q: Why should you avoid infinite retries on database errors?**
**A:** Infinite retries can create a "retry storm," where thousands of requests keep hammering a database that is already down, preventing it from ever recovering. Always use limited retries and exponential backoff.
