---
---

# Write-Behind (Write-Back)

In the **Write-Behind** strategy, data is written to the cache first, and the write to the database happens **asynchronously** after a certain delay or once enough data has accumulated.

## **How it Works**
1. **Write Request**:
   - The application writes data to the cache.
   - The cache immediately acknowledges the write to the application.
2. **Asynchronous Write**:
   - A background process or a worker eventually pushes the cached data to the database.
   - Writes can be batched together to improve database performance.

## **Pros**
- **Extreme Write Performance**: Writes are limited only by the speed of the cache (RAM), not the database (Disk).
- **Reduced DB Load**: Batching multiple writes into a single database transaction reduces the overhead on the DB.
- **Improved Availability**: The system can continue to accept writes even if the database is temporarily down.

## **Cons**
- **Data Loss Risk**: If the cache crashes before the data is flushed to the database, that data is lost permanently.
- **Complexity**: Implementing reliable asynchronous flushes, handling failures, and ensuring eventual consistency is difficult.
- **Inconsistency**: For a short period, the database contains outdated information compared to the cache.

## **Go Context: Implementation**

This pattern often involves a producer-consumer model using Go channels or a message queue.

```go
type WriteBehindCache struct {
    data  sync.Map
    queue chan User
    db    *sql.DB
}

func (c *WriteBehindCache) Save(user User) {
    // 1. Write to Cache (Instant)
    c.data.Store(user.ID, user)

    // 2. Queue for async flush
    c.queue <- user
}

func (c *WriteBehindCache) flushWorker() {
    var batch []User
    for {
        select {
        case user := <-c.queue:
            batch = append(batch, user)
            if len(batch) >= 100 {
                c.writeToDB(batch)
                batch = nil
            }
        case <-time.After(5 * time.Second):
            if len(batch) > 0 {
                c.writeToDB(batch)
                batch = nil
            }
        }
    }
}
```

## **Best Practices**
- Use only for non-critical data where high write throughput is more important than durability (e.g., analytics, real-time counters).
- Implement persistent queues (like Kafka or RabbitMQ) instead of in-memory Go channels to reduce data loss risk.
- Monitor the queue depth to ensure the flusher isn't falling behind.
