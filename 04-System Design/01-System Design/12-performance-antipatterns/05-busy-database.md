---
---

# Busy Database

## Summary
Offloading excessive computation, formatting, or business logic to the database server. This makes the database a bottleneck, as scaling a database horizontally is much harder and more expensive than scaling application servers (compute).

## Detailed Development
Databases are optimized for data storage, retrieval, and aggregation (e.g., `SUM`, `AVG`). They are **not** optimized for:
- **Formatting**: Converting dates, currency, or generating JSON/XML strings (e.g., `FOR XML` in SQL Server).
- **String Manipulation**: Complex regex or parsing long strings.
- **Business Logic**: Large stored procedures that contain core application rules.
- **Heavy Sorting/Filtering**: Performing complex sorts that could be done in the app tier if the dataset is small enough.

### Why it's a problem
1. **Cost**: Managed database units (like Azure DTUs or AWS RDS instances) are more expensive than general-purpose compute (EC2/Lambda).
2. **Scalability**: You can easily spin up 100 Go microservice instances, but you usually have only one primary database.
3. **Debugging**: Debugging a 500-line stored procedure is significantly harder than debugging Go code with standard tooling.

## Go-Specific Application

### 1. Move Formatting to Go
Instead of asking SQL to format a date or currency, retrieve the raw data and use Go's `time` and `fmt` packages.

```go
// BAD: SQL does the work
rows, _ := db.Query("SELECT FORMAT(created_at, 'yyyy-MM-dd') FROM orders")

// GOOD: Go does the work
var createdAt time.Time
db.QueryRow("SELECT created_at FROM orders").Scan(&createdAt)
formatted := createdAt.Format("2006-01-02")
```

### 2. JSON Processing
Avoid using DB-specific JSON functions for complex logic. Fetch the JSON blob and use Go's `encoding/json` or `github.com/tidwall/gjson` for high-performance parsing.

```go
// Retrieve the raw JSONB from Postgres
var data []byte
db.QueryRow("SELECT metadata FROM events WHERE id = 1").Scan(&data)

// Process in Go
var meta Metadata
json.Unmarshal(data, &meta)
// perform complex logic here...
```

### 3. Aggregation vs. Application Logic
- **Use Database for**: `SELECT COUNT(*)`, `SUM(amount) WHERE user_id = ?`.
- **Use Go for**: Calculating complex insurance premiums based on 50 different retrieved fields.

## Interview Preparation

### Questions
1. **When should you use a Stored Procedure?**
   - When you need to perform multiple dependent operations and want to avoid network round-trips (though a Transaction in Go can often achieve the same).
   - When strict data integrity must be enforced at the DB level regardless of which application connects.

2. **Is it always better to sort in the application?**
   - No. If you have 1 million rows and only need the top 10, the database **must** do the sorting using its indexes. If you have 50 rows already fetched, sorting them in Go is faster and cheaper.

3. **What is the "Repository Pattern"'s role in avoiding a Busy Database?**
   - It abstracts data access, allowing you to easily move logic from SQL queries into Go methods without changing the service layer.
