# SQLx
---
---

## Summary
SQLx is a modern, asynchronous SQL toolkit for Rust. Unlike an ORM, it doesn't try to hide the SQL. Instead, it embraces it. Its standout feature is the ability to check your raw SQL queries for syntax and type correctness at compile time by connecting to a live database during the build process.

## Detailed Explanation

### Core Philosophy
"Pure Rust, Async, and Raw SQL." SQLx is designed for developers who love SQL and want full control over their queries but still want the safety guarantees of Rust. It avoids the complexity of DSLs (like Diesel) and works directly with `tokio` or `async-std`.

### Key Features
*   **Compile-Time Verification**: The `query!` macro connects to your DB (or a saved offline data file) to verify SQL syntax and types.
*   **Async Native**: Built from the ground up for asynchronous usage.
*   **Database Agnostic**: Supports PostgreSQL, MySQL, SQLite, and MSSQL with a unified API where possible.
*   **Offline Mode**: Allows CI/CD builds to verify queries without a running database using a `sqlx-data.json` file.

### Use Cases
*   **High-Performance Backends**: Where async I/O is crucial for throughput.
*   **Complex Queries**: When ORMs struggle to represent complex joins or window functions.
*   **Microservices**: Lightweight and fast.

### Code Example
*Dependencies: `sqlx`, `tokio`*

```rust
use sqlx::postgres::PgPoolOptions;

#[derive(sqlx::FromRow, Debug)]
struct User {
    id: i32,
    username: String,
}

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect("postgres://user:pass@localhost/db").await?;

    // This query is checked at compile time!
    let users = sqlx::query_as!(
        User,
        "SELECT id, username FROM users WHERE active = $1",
        true
    )
    .fetch_all(&pool)
    .await?;

    for user in users {
        println!("{:?}", user);
    }

    Ok(())
}
```

## Interview Questions

1.  **Q: How does the `query!` macro work in SQLx?**
    *   **A:** The `query!` macro expands at compile time. It connects to the database URL specified in your `.env` file, prepares the SQL statement to check for syntax errors, and describes the result columns to ensure they match the Rust types you are assigning them to.

2.  **Q: What is "Offline Mode" in SQLx and why is it important?**
    *   **A:** Offline Mode allows you to compile your project without connecting to a live database. This is critical for CI/CD pipelines or sharing code with others who don't have the database running. It works by "caching" the query metadata into a `sqlx-data.json` file using the `cargo sqlx prepare` command.

3.  **Q: Can SQLx be used synchronously?**
    *   **A:** No, SQLx is purely asynchronous. If you need a synchronous/blocking driver (e.g., for a simple CLI tool or legacy codebase), you should look at `rusqlite` or `diesel`.
