---
---

# Write-Through Cache

In the **Write-Through** strategy, the application treats the cache as the primary data store. When data is written, it is written to the cache and the database **simultaneously** and **synchronously**.

## **How it Works**
1. **Write Request**:
   - The application writes data to the cache.
   - The cache synchronously writes the data to the database.
   - The operation is only considered successful once both writes complete.
2. **Read Request**:
   - Similar to Cache-Aside, the app checks the cache.
   - Since data is written to the cache during every write, the data is almost always available in the cache (Cache Hit).

## **Pros**
- **Data Consistency**: The cache and database are always in sync. There is no risk of stale data.
- **Read Performance**: Since data is populated in the cache during writes, subsequent reads are very fast (no initial cache miss penalty).

## **Cons**
- **Write Latency**: Every write operation must wait for both the cache and the database to finish. This increases overall latency for write-heavy applications.
- **Cache Pollution**: Data that is rarely read but frequently written will still occupy space in the cache, potentially pushing out more valuable data.

## **Go Context: Implementation**

In Go, this is usually implemented by wrapping the database and cache operations within a single method.

```go
type Repository struct {
    db    *sql.DB
    redis *redis.Client
}

func (r *Repository) SaveUser(ctx context.Context, user User) error {
    // 1. Write to Cache
    data, _ := json.Marshal(user)
    if err := r.redis.Set(ctx, "user:"+user.ID, data, 0).Err(); err != nil {
        return fmt.Errorf("cache write failed: %w", err)
    }

    // 2. Write to DB (Synchronously)
    _, err := r.db.ExecContext(ctx, "INSERT INTO users ...", user.Name, user.ID)
    if err != nil {
        // Rollback cache if DB fails to maintain consistency? 
        // This is tricky and often requires distributed transactions or compensating logic.
        r.redis.Del(ctx, "user:"+user.ID)
        return fmt.Errorf("db write failed: %w", err)
    }

    return nil
}
```

## **Best Practices**
- Use Write-Through when data consistency is critical and you can afford the extra write latency.
- Combine with an eviction policy (like LRU) to prevent cache pollution.
- Often used in conjunction with a **Read-Through** pattern where the cache library handles the DB sync internally.
