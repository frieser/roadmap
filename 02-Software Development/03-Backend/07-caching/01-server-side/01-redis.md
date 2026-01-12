---
---

## Summary
Redis (Remote Dictionary Server) is an open-source, in-memory data structure store, used as a database, cache, and message broker. It supports data structures such as strings, hashes, lists, sets, and sorted sets with range queries.

## Detailed Explanation
Redis is the most popular choice for server-side caching due to its speed (everything is in memory) and versatility.

### Key Features
- **In-Memory Storage**: Provides extremely low latency (sub-millisecond) for read and write operations.
- **Persistence**: Unlike some caches, Redis can optionally persist data to disk (RDB snapshots or AOF logs).
- **Data Structures**: Not just simple key-value pairs; you can store complex objects like hashes or lists.
- **Eviction Policies**: Automatically removes old data when memory is full (e.g., Least Recently Used - LRU).
- **Replication & Sentinel**: High availability through primary-replica setups and automatic failover.

### Common Caching Patterns
- **Cache-Aside**: Application checks the cache; if not found, it reads from the DB and updates the cache.
- **Write-Through**: Application writes to the cache, and the cache synchronously updates the DB.
- **Write-Behind (Write-Back)**: Application writes to the cache, which then asynchronously updates the DB.

## Go Context
In Go, `go-redis/redis` (v8/v9) is the most widely used client.

### Example: Simple Redis Cache in Go
```go
package main

import (
	"context"
	"fmt"
	"time"
	"github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
	rdb := redis.NewClient(&redis.Options{
		Addr: "localhost:6379",
	})

	// Set a value with 10 minute expiration
	err := rdb.Set(ctx, "user:123", "John Doe", 10*time.Minute).Err()
	if err != nil {
		panic(err)
	}

	// Get a value
	val, err := rdb.Get(ctx, "user:123").Result()
	if err == redis.Nil {
		fmt.Println("Key does not exist")
	} else if err != nil {
		panic(err)
	} else {
		fmt.Println("Value:", val)
	}
}
```

## Interview Questions
- **Q: What is a Cache Stampede and how do you prevent it?**
- **A:** It occurs when many requests for a popular but expired cache key hit the database simultaneously. It can be prevented using locking (only one request fetches from DB while others wait) or by setting a "soft" expiration that refreshes the cache in the background before it actually expires.

- **Q: Redis vs. Memcached: When would you choose Redis?**
- **A:** Choose Redis if you need persistence, complex data structures (like sorted sets), built-in replication, or pub/sub capabilities.

- **Q: How does Redis handle memory management?**
- **A:** Redis uses eviction policies like `allkeys-lru` or `volatile-lru`. You can set a `maxmemory` limit, and Redis will discard keys according to the chosen policy when that limit is reached.
