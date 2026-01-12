---
---

## Summary
**Denormalization** is the optimization technique of storing redundant data in a database to improve read performance. By duplicating data (e.g., storing `user_name` in the `comments` table), you avoid expensive `JOIN` operations at the cost of slower writes and increased storage.

## Detailed Explanation

### Normalization vs. Denormalization
*   **Normalization (3NF)**: Minimizes redundancy. "Update in one place." Best for data integrity and write performance.
*   **Denormalization**: Maximizes read speed. Pre-calculates joins and aggregates. Best for read-heavy workloads.

### Common Techniques
1.  **Pre-joining Tables**: Adding columns from a related table to the main table.
    *   *Example*: Adding `author_name` to the `posts` table so you don't need to join `users`.
2.  **Summary Tables**: Storing aggregated data.
    *   *Example*: A `daily_sales` table that stores the sum of `orders` for each day, updated nightly or via triggers.
3.  **Cached Counters**: Storing `likes_count` on the `post` table instead of running `SELECT COUNT(*) FROM likes` every time.

### Trade-offs
*   **Write Complexity**: Every time the source data changes (e.g., user changes name), you must update all redundant copies.
*   **Inconsistency Risk**: If a sync fails, users might see conflicting data (e.g., Profile shows "Jane", Comment shows "John").

## Go Context: Keeping Data in Sync
Using a background worker or event bus to update denormalized fields.

```go
func UpdateUserProfile(userID int, newName string) {
	// 1. Transactional update to source of truth
	tx, _ := db.Begin()
	tx.Exec("UPDATE users SET name=? WHERE id=?", newName, userID)
	tx.Commit()

	// 2. Async update to denormalized tables (Eventual Consistency)
	go func() {
		// Use a separate connection/service
		db.Exec("UPDATE comments SET author_name=? WHERE author_id=?", newName, userID)
		db.Exec("UPDATE posts SET author_name=? WHERE author_id=?", newName, userID)
	}()
}
```

## Interview Questions

### Q: When is denormalization the right choice?
**A:** When you have a massive Read/Write ratio (e.g., 1000:1) and `JOIN`s are the bottleneck. It is commonly used in data warehouses (OLAP) and high-scale web apps (NoSQL data modeling heavily relies on this).

### Q: What is a Materialized View?
**A:** A database object that stores the result of a query physically. It is a form of managed denormalization. The database handles the "refresh" of this data (either automatically on commit or periodically), providing fast reads of complex aggregations.
