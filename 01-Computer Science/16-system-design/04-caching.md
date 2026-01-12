---
---

## Summary
**Caching** is the temporary storage of data to reduce latency and database load. It stores copies of frequently accessed data in fast memory (RAM) closer to the application.

## Detailed Explanation
### Caching Strategies
1.  **Cache-Aside (Lazy Loading)**: App checks Cache. If miss, App reads DB and updates Cache. (Most common).
2.  **Write-Through**: App writes to Cache, Cache writes to DB synchronously. (Data safety, slower write).
3.  **Write-Back (Write-Behind)**: App writes to Cache. Cache writes to DB asynchronously. (Fast write, risk of data loss).
4.  **Write-Around**: App writes directly to DB (skips cache). Cache is only populated on Read miss. (Good for "write-once, read-rarely" data).

### Eviction Policies
*   **LRU (Least Recently Used)**: Remove the item accessed longest ago.
*   **LFU (Least Frequently Used)**: Remove the item accessed fewest times.
*   **TTL (Time To Live)**: Auto-expire keys after X seconds.

### Go Context
In-memory caching is easy in Go (using `map` + `sync.RWMutex`), but usually, we use distributed caches like **Redis**.

```go
package main

import "sync"

type MemoryCache struct {
	store map[string]string
	lock  sync.RWMutex
}

func (c *MemoryCache) Get(key string) (string, bool) {
	c.lock.RLock()
	defer c.lock.RUnlock()
	val, ok := c.store[key]
	return val, ok
}

func (c *MemoryCache) Set(key, val string) {
	c.lock.Lock()
	defer c.lock.Unlock()
	c.store[key] = val
}
```

## Interview Questions
**Q: What is the "Thundering Herd" problem?**
A: When a popular cache key expires, thousands of concurrent requests all miss the cache at the same moment and hit the database simultaneously, causing it to crash. Solution: Locking/Mutex on cache miss, or random jitter on TTL.

**Q: Where can you place a cache?**
A: Browser, CDN, Reverse Proxy, App Memory (Local Cache), Distributed Cache (Redis), Database Buffer Pool.

## Diagram
```mermaid
graph LR
    App -->|1. Get Key| Cache
    Cache --|2. Miss| App
    App -->|3. Read| DB
    DB -->|4. Return| App
    App -->|5. Set Key| Cache
```
