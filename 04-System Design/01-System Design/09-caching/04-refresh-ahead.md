---
---

# Refresh-Ahead

**Refresh-Ahead** is a strategy where the cache proactively refreshes its data before it is predicted to expire. This ensures that popular data is always available and up-to-date in the cache, eliminating the latency penalty of a cache miss.

## **How it Works**
1. **Prediction**: The cache tracks the usage patterns and expiration times of its entries.
2. **Proactive Fetch**: Before an entry expires (e.g., at 90% of its TTL), if it has been recently accessed, the cache automatically fetches the latest data from the database in the background.
3. **Seamless Transition**: The new data replaces the old data just as it was about to expire.

## **Pros**
- **Eliminates Latency**: For frequently accessed items, users never experience a cache miss because the data is refreshed before it "goes cold."
- **Predictable Performance**: System response times stay consistent even for hot keys.

## **Cons**
- **Extra Database Load**: If the prediction is wrong, the system might refresh data that no one ends up reading, wasting DB resources.
- **Complexity**: Requires sophisticated tracking of TTLs and access frequencies to decide what to refresh.

## **Go Context: Implementation**

In Go, this can be implemented using a background goroutine that periodically checks TTLs and re-fetches data.

```go
func (c *Cache) RefreshAhead(ctx context.Context, key string) {
    ticker := time.NewTicker(time.Minute * 9) // Refresh every 9 mins for a 10 min TTL
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            // Fetch fresh data from DB
            newData, err := c.db.Fetch(key)
            if err == nil {
                c.store.Set(key, newData, time.Minute*10)
            }
        case <-ctx.Done():
            return
        }
    }
}
```

## **Best Practices**
- Use only for **hot keys** (frequently accessed data).
- Ensure the re-fetch logic doesn't overload the database during peak times (use rate limiting or jitter).
- CDNs often use a variant of this called **Stale-While-Revalidate**, where they serve the stale data once while fetching the fresh version in the background.
