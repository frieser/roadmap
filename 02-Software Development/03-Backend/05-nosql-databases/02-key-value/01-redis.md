---
---

# Redis (Remote Dictionary Server)

Redis is an open-source, in-memory data structure store, used as a database, cache, and message broker. It is a fundamental tool for senior backend developers to master due to its versatility and performance.

## 1. Single-Threaded Architecture

Despite being primarily single-threaded, Redis achieves high performance (100k+ ops/sec) through:
- **Non-blocking I/O**: Uses I/O multiplexing (e.g., `epoll` on Linux, `kqueue` on BSD) to handle thousands of concurrent connections.
- **In-Memory Operations**: Data is stored in RAM, eliminating disk seek times.
- **Efficiency**: No context switching or locking overhead between threads.

**Note on Redis 6+**: Introduced **I/O Threads** to handle network reading/writing in parallel, but command execution remains single-threaded.

**Evidence ([source](https://github.com/redis/redis/blob/unstable/src/ae.c#L492-L498)):**
```c
void aeMain(aeEventLoop *eventLoop) {
    eventLoop->stop = 0;
    while (!eventLoop->stop) {
        aeProcessEvents(eventLoop, AE_ALL_EVENTS|
                                   AE_CALL_BEFORE_SLEEP|
                                   AE_CALL_AFTER_SLEEP);
    }
}
```

## 2. Advanced Data Structures

- **Sorted Sets (ZSET)**: Uses a **Skip List** and a Hash Table. Allows $O(\log N)$ addition, removal, and range queries.
- **HyperLogLog**: Probabilistic data structure used to estimate cardinality (unique elements) with fixed memory (approx. 12KB) and 0.81% error rate.
- **Streams**: An append-only log data structure with consumer groups, similar to Kafka. Supports `XADD`, `XREAD`, and `XGROUP`.

## 3. Persistence Models

- **RDB (Redis Database)**: Point-in-time binary snapshots.
    - *Pros*: Fast restarts, compact file.
    - *Cons*: Data loss between snapshots if Redis crashes.
- **AOF (Append Only File)**: Logs every write operation.
    - *Pros*: High durability (can be configured to `fsync` every second).
    - *Cons*: Large file size, slower restarts.
- **Hybrid**: Many production setups use both (RDB for fast restart + AOF for durability).

## 4. Clustering and High Availability

- **Redis Sentinel**: Provides high availability (monitoring, notification, automatic failover) but does not provide sharding.
- **Redis Cluster**: Provides sharding (distributing data across nodes) and high availability.
    - Uses **16,384 Hash Slots** to distribute keys.
    - Clients are redirected to the correct node if they query the wrong one (MOVED redirection).

## 5. Eviction Policies

When `maxmemory` is reached, Redis uses eviction policies to free up space:
- `allkeys-lru`: Evicts the least recently used keys regardless of expiration.
- `volatile-lru`: Evicts LRU keys that have an expiration set.
- `allkeys-lfu`: Evicts the least frequently used keys.
- `volatile-ttl`: Evicts keys with the shortest Time-To-Live.

## 6. Common Caching Patterns

- **Cache-Aside (Lazy Loading)**: Application checks the cache; if not found, it reads from the DB and updates the cache. Most common pattern.
- **Write-Through**: Application writes to the cache, and the cache synchronously updates the DB.
- **Write-Behind (Write-Back)**: Application writes to the cache, which then asynchronously updates the DB (good for write-heavy workloads).

## 7. Go Integration (go-redis)

The standard client for Go is `github.com/redis/go-redis/v9`.

### Example: Redis Streams in Go
```go
import (
    "context"
    "github.com/redis/go-redis/v9"
)

func main() {
    rdb := redis.NewClient(&redis.Options{Addr: "localhost:6379"})
    ctx := context.Background()

    // Add an entry to a stream
    err := rdb.XAdd(ctx, &redis.XAddArgs{
        Stream: "orders",
        Values: map[string]interface{}{"order_id": "123", "status": "pending"},
    }).Err()
}
```

## Interview Questions (Senior Level)

1. **Q: How does Redis handle "Cache Stampede"?**
   - **A**: Use locks (SETNX) so only one process fetches data from the DB, or use probabilistic early expiration (recompute before it expires).
2. **Q: Redis Cluster vs Sentinel?**
   - **A**: Sentinel is for HA of a single primary-replica set. Cluster is for horizontal scaling (sharding) + HA.
3. **Q: Why is Redis single-threaded?**
   - **A**: Memory is the bottleneck, not CPU. Single-threading avoids synchronization overhead and simplifies implementation.
