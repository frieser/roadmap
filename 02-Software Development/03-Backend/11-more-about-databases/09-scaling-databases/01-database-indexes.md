---
---

## Summary
A Database Index is a data structure that improves the speed of data retrieval operations on a database table. It acts like an index in a book, allowing the database to find records without scanning the entire table.

## Detailed Explanation
Indices are essential for performance as data grows. However, they come with a cost: they occupy disk space and slow down write operations (Insert, Update, Delete) because the index must be updated every time the data changes.

### Types of Indexes
-   **B-Tree (default)**: Efficient for equality and range queries.
-   **Hash**: Fast for exact matches but doesn't support ranges.
-   **GIN / GiST**: Used for complex data types like JSONB or full-text search.
-   **Composite Index**: An index on multiple columns. Order matters!

### Best Practices
-   Index columns used in `WHERE`, `JOIN`, and `ORDER BY`.
-   Avoid indexing columns with low cardinality (e.g., a boolean "is_active").
-   Use Composite Indexes for queries that frequently filter by multiple columns.

## Go-specific Context
In Go, you typically define indexes via your ORM models or your SQL migration files.

### Defining Indexes in GORM
```go
type Product struct {
    ID    uint
    Code  string `gorm:"index"` // Basic index
    Price uint   `gorm:"index:idx_price,priority:1"` // Named index
    Name  string `gorm:"uniqueIndex"` // Unique index
}
```

### Composite Index in GORM
```go
type User struct {
    ID        uint
    FirstName string `gorm:"index:idx_full_name"`
    LastName  string `gorm:"index:idx_full_name"` // Composite index on both
}
```

## Interview Questions
**Q: Why does the order of columns in a Composite Index matter?**
**A:** Because B-Tree indexes are sorted. An index on `(A, B)` can be used for queries on `A` or `(A, B)`, but it cannot be used efficiently for a query that only filters by `B`. This is known as the **Left-Prefix Rule**.

**Q: What is a 'Covering Index'?**
**A:** A covering index is an index that contains all the data required for a query. If the index "covers" the query, the database doesn't need to look at the actual table rows at all, making the query extremely fast.

**Q: Can having too many indexes be a bad thing?**
**A:** Yes. Every index slows down write operations and consumes disk space. You should only add indexes that are actually used by your queries.
