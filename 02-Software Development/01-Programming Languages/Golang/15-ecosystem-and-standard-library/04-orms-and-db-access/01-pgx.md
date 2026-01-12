# PGX (PostgreSQL Driver)

## Summary
`pgx` is a pure Go driver and toolkit for PostgreSQL. It is significantly faster and more feature-rich than the standard `lib/pq` driver (which is effectively in maintenance mode). `pgx` supports PostgreSQL-specific features like Listen/Notify, COPY, large objects, and custom binary serialization, and includes a high-performance connection pool (`pgxpool`).

## Detailed Explanation

### 1. Why `pgx`?
*   **Performance**: It uses the PostgreSQL binary format for data transfer, avoiding expensive text parsing.
*   **Features**: Supports Arrays, JSON/JSONB, and composite types natively.
*   **Maintenance**: It is the actively developed, de-facto standard driver for Go projects today.

### 2. Connection Pooling (`pgxpool`)
Unlike `lib/pq` which relies on `database/sql` for pooling, `pgx` provides its own pool implementation that is optimized for high concurrency.

```go
package main

import (
	"context"
	"fmt"
	"os"

	"github.com/jackc/pgx/v5/pgxpool"
)

func main() {
	ctx := context.Background()
	
    // Connect using connection string
	dbpool, err := pgxpool.New(ctx, "postgres://user:pass@localhost:5432/dbname")
	if err != nil {
		fmt.Fprintf(os.Stderr, "Unable to create connection pool: %v\n", err)
		os.Exit(1)
	}
	defer dbpool.Close()

    // QueryRow (Read)
	var name string
	var weight int64
	err = dbpool.QueryRow(ctx, "SELECT name, weight FROM widgets WHERE id=$1", 42).Scan(&name, &weight)
    
    // Exec (Write)
    commandTag, err := dbpool.Exec(ctx, "UPDATE widgets SET weight=$1 WHERE id=$2", weight+1, 42)
}
```

### 3. Custom Types
`pgx` allows you to map Go structs directly to Postgres composite types or JSON columns by implementing the `EncodeBinary` and `DecodeBinary` interfaces, or simply using `pgx.RowToStructByName` helpers.

## Interview Questions

**Q: What is the main performance advantage of `pgx` over `lib/pq`?**
**A:** `pgx` uses the PostgreSQL **binary protocol** for transmitting values, whereas `lib/pq` uses the text protocol. Parsing binary integers, floats, and timestamps is much faster for the CPU than parsing ASCII strings. `pgx` also allows for batched queries (pipelining), reducing network round-trips.

**Q: Should I use `pgx` with `database/sql` or the `pgx` native interface?**
**A:** It depends. The `pgx` native interface (used via `pgxpool` or `pgx.Conn`) gives you access to PostgreSQL-specific features (COPY, Listen/Notify) and better performance. Using `database/sql` (via the `pgx` stdlib adapter) makes your code portable to other databases (like MySQL) but hides the advanced capabilities of Postgres. New Go projects using Postgres exclusively should prefer the native interface.

**Q: How does `pgx` handle NULL values?**
**A:** `pgx` supports standard `sql.NullString` types but also provides its own generic-friendly types or simply allows scanning into pointers (`*string`). If the DB value is NULL, the pointer is set to `nil`.
