# Rusqlite
---
---

## Summary
Rusqlite provides ergonomic, synchronous bindings to SQLite. It is the standard choice for applications that need an embedded database without the overhead of an external server. It is lightweight, fast, and wraps the C SQLite API in safe Rust abstractions.

## Detailed Explanation

### Core Philosophy
Rusqlite aims to be the most direct and idiomatic way to use SQLite in Rust. It doesn't force an async runtime on you (since SQLite is fundamentally a blocking file-system based DB) and doesn't impose a heavy ORM layer.

### Key Features
*   **Embedded**: No separate server process; the DB is just a file.
*   **Bundled**: Can compile the C SQLite library from source, simplifying deployment (no system dependencies needed).
*   **Safe Interface**: Handles memory management and error checking for the underlying C API.

### Use Cases
*   **CLI Tools**: Storing history, cache, or configuration.
*   **Desktop Applications**: Local data storage for apps (like a notes app).
*   **Edge/IoT**: Lightweight storage on resource-constrained devices.

### Code Example
*Dependencies: `rusqlite`*

```rust
use rusqlite::{params, Connection, Result};

#[derive(Debug)]
struct Cat {
    id: i32,
    name: String,
    color: String,
}

fn main() -> Result<()> {
    let conn = Connection::open("cats.db")?;

    conn.execute(
        "CREATE TABLE IF NOT EXISTS cats (
            id    INTEGER PRIMARY KEY,
            name  TEXT NOT NULL,
            color TEXT NOT NULL
        )",
        [],
    )?;

    let meow = Cat {
        id: 0,
        name: "Rusty".to_string(),
        color: "Orange".to_string(),
    };

    conn.execute(
        "INSERT INTO cats (name, color) VALUES (?1, ?2)",
        params![meow.name, meow.color],
    )?;

    let mut stmt = conn.prepare("SELECT id, name, color FROM cats")?;
    let cats_iter = stmt.query_map([], |row| {
        Ok(Cat {
            id: row.get(0)?,
            name: row.get(1)?,
            color: row.get(2)?,
        })
    })?;

    for cat in cats_iter {
        println!("Found cat: {:?}", cat.unwrap());
    }
    Ok(())
}
```

## Interview Questions

1.  **Q: Why is Rusqlite synchronous/blocking?**
    *   **A:** SQLite operates directly on disk files. Most filesystem operations are blocking by nature. While async file I/O exists, the overhead of context switching often outweighs the benefits for local embedded DBs. If you need to use it in an async context (like Tokio), you should wrap it in `task::spawn_blocking`.

2.  **Q: What is the benefit of the `bundled` feature in Rusqlite?**
    *   **A:** Enabling the `bundled` feature flag tells Cargo to compile a specific version of SQLite from source and link it statically. This ensures your app works on any system without needing `libsqlite3` installed and guarantees you are using a known, tested version of SQLite.

3.  **Q: How do you handle migrations with Rusqlite?**
    *   **A:** Rusqlite doesn't have a built-in migration system like Diesel or SQLx. Developers typically use a separate crate like `refinery` or write simple logic to check the `user_version` PRAGMA and apply schema updates sequentially on startup.
