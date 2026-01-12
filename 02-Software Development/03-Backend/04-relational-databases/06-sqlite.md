---
---

## Summary
SQLite is a C-language library that implements a small, fast, self-contained, high-reliability, full-featured, SQL database engine. Unlike most other SQL databases, SQLite does not have a separate server process. It reads and writes directly to ordinary disk files.

## Detailed Explanation

### Architecture
SQLite's architecture is unique because it is **embedded** within the application process.
- **Interface**: The C API used by the application.
- **Compiler**: Tokenizer, Parser, and Code Generator that turns SQL into **Bytecode**.
- **Virtual Machine**: Executes the bytecode to perform database operations.
- **Backend**: Uses B-Trees for storage and a Pager to manage the file system interface and transactions.
- **WAL Mode**: Write-Ahead Logging allows for multiple readers to co-exist with a single writer, significantly improving concurrency.

### Pros and Cons
| Pros | Cons |
| :--- | :--- |
| **Zero Configuration**: No server to install, setup, or manage. | **Concurrency**: Only one process can write to the database at a time. |
| **Portability**: The entire database is a single file on disk. | **Scale**: Not suitable for high-traffic client/server web applications. |
| **Reliability**: Extremely well-tested (100% branch test coverage). | **Features**: No built-in user management or stored procedures. |

### When to Use
- Local data storage for mobile apps (iOS/Android) and desktop apps.
- Edge computing and IoT devices.
- Caching layer for expensive API calls.
- Automated testing (using in-memory databases).
- Low-to-medium traffic websites (e.g., blogs).

### Go-Specific Context
Go has excellent support for SQLite, but there is a major split between CGO and CGO-free drivers.
- **CGO Driver**: `github.com/mattn/go-sqlite3`. The most mature and feature-rich, but requires a C compiler and makes cross-compilation difficult.
- **CGO-Free Driver**: `modernc.org/sqlite`. A pure Go port created by transpiling the SQLite C source code to Go. It allows for easy cross-compilation.
- **Connection Tuning**: Because SQLite allows only one writer, it is crucial to set `db.SetMaxOpenConns(1)` for the writer instance to avoid "database is locked" errors.

#### Example: CGO-Free Connection
```go
package main

import (
	"database/sql"
	"fmt"
	"log"

	_ "modernc.org/sqlite"
)

func main() {
	// "file:demo.db" or ":memory:" for in-memory
	db, err := sql.Open("sqlite", "demo.db")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	// Essential for SQLite to avoid lock contention in some modes
	db.SetMaxOpenConns(1)

	_, err = db.Exec("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)")
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("SQLite is ready!")
}
```

## Interview Questions

**Q: Is SQLite a "real" database?**
**A:** Yes. SQLite is a full-featured, ACID-compliant relational database. It supports most of the SQL standard, including transactions, joins, subqueries, and views. Its primary difference is that it is a library, not a server.

**Q: What is WAL mode in SQLite?**
**A:** WAL (Write-Ahead Logging) is a journaling mode that allows multiple readers to read the database at the same time a writer is modifying it. In older modes (like DELETE), the writer would lock the entire database, blocking readers. WAL provides better concurrency and is generally faster.

**Q: Why would you use `modernc.org/sqlite` instead of `mattn/go-sqlite3`?**
**A:** You would use `modernc.org/sqlite` if you want to avoid CGO dependencies. This makes the build process faster and allows you to cross-compile your Go binary (e.g., building a Linux binary on macOS) without needing a cross-C-compiler. It simplifies the development environment significantly.
