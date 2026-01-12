---
---

## Summary
**SQL Tuning** is the process of optimizing database queries and schema design to improve performance and resource utilization. It involves analyzing execution plans, creating appropriate indexes, and rewriting inefficient queries to ensure the database engine retrieves data via the most efficient path.

## Detailed Explanation

### 1. Indexing Strategies
*   **B-Tree Indexes**: The default for most RDBMS. Great for equality (`=`) and range (`<`, `>`, `BETWEEN`) queries.
*   **Composite Indexes**: Indexes on multiple columns. Order matters (Leftmost Prefix Rule).
    *   *Index(A, B)* supports queries on `A` and `A AND B`, but NOT on `B` alone.
*   **Covering Indexes**: An index that contains all the fields required by the query, allowing the DB to skip reading the actual table rows ("Index Only Scan").

### 2. The `EXPLAIN` Command
Always analyze slow queries using `EXPLAIN` (or `EXPLAIN ANALYZE` in Postgres).
*   **Sequential Scan (Full Table Scan)**: Reading every row. Expensive for large tables.
*   **Index Scan**: Using the index tree. efficient.
*   **Filesort**: Sorting rows on disk because memory was insufficient.

### 3. Common Anti-Patterns
*   **N+1 Problem**: Fetching a list of items (1 query) and then looping to fetch details for each (N queries). **Fix**: Use `JOIN` or `IN (...)`.
*   **`SELECT *`**: Fetching unnecessary columns increases network I/O and prevents covering indexes.
*   **Functions on Columns**: `WHERE YEAR(created_at) = 2023` prevents index usage. **Fix**: `WHERE created_at BETWEEN '2023-01-01' AND '2023-12-31'`.

## Go Example: Debugging Queries

```go
package main

import (
	"database/sql"
	"fmt"
	"log"
)

func AnalyzeQuery(db *sql.DB, userID int) {
	// Good practice: Use placeholders to prevent SQL Injection and allow plan caching
	query := "EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = $1"
	
	rows, err := db.Query(query, userID)
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()

	for rows.Next() {
		var plan string
		rows.Scan(&plan)
		fmt.Println(plan)
	}
}
```

## Interview Questions

### Q: What is the "Leftmost Prefix" rule?
**A:** In a composite index (e.g., on columns A, B, C), the index can be used for searches on A, or A and B, or A and B and C. It cannot be used for searches on B or C alone, or B and C, because the tree is sorted first by A.

### Q: Why does adding too many indexes hurt performance?
**A:** Every time you `INSERT`, `UPDATE`, or `DELETE` a row, the database must update all corresponding indexes. This adds significant overhead to write operations. Indexes trade write speed for read speed.
