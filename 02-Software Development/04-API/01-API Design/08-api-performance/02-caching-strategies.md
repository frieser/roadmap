# Caching Strategies

## Summary
Caching is a technique to store copies of frequently accessed data in a temporary storage location (cache) to reduce latency and backend load. Effective caching strategies involve multiple layers: **Client-side** (browser), **CDN** (edge), and **Server-side** (application/database). Mastering **Cache Invalidation** and **Conditional Requests** (ETags) is essential for maintaining data consistency while maximizing performance.

## Detailed Explanation

### Layers of Caching

1.  **Client-Side Caching**:
    *   Controlled by HTTP headers sent by the server.
    *   **Headers**: `Cache-Control` (max-age, private, public), `Expires`.
    *   *Benefit*: Zero network latency for the client.

2.  **CDN (Content Delivery Network) Caching**:
    *   Caches content at the "edge" (servers geographically closer to the user).
    *   Great for static assets (images, CSS) and public API responses.
    *   *Benefit*: Reduces load on origin servers and lowers latency for global users.

3.  **Server-Side Caching**:
    *   **In-Memory (Local)**: Fast but stateful. Good for read-heavy, low-consistency data. (e.g., `sync.Map` or local variable).
    *   **Distributed**: External store accessible by all instances. (e.g., Redis, Memcached). Essential for horizontal scaling.

4.  **Database Caching**:
    *   Query caching within the database engine itself or application-level caching of query results.

### Caching Strategies
*   **Cache-Aside (Lazy Loading)**: App checks cache; if miss, reads DB and updates cache. Most common.
*   **Write-Through**: App updates DB and cache simultaneously. Ensures consistency but slower writes.
*   **Write-Back (Write-Behind)**: App updates cache; cache updates DB asynchronously. Fast writes, risk of data loss.

### Conditional Requests
Mechanisms to validate if a cached resource is still fresh.
*   **ETag (Entity Tag)**: Unique fingerprint of the resource. Client sends `If-None-Match: "tag"`. Server returns `304 Not Modified` if match.
*   **Last-Modified**: Timestamp. Client sends `If-Modified-Since`.

### Go Implementation: Redis Cache-Aside
Using `github.com/redis/go-redis/v9`.

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"time"

	"github.com/redis/go-redis/v9"
)

type User struct {
	ID    string `json:"id"`
	Name  string `json:"name"`
	Email string `json:"email"`
}

var ctx = context.Background()

func getUser(rdb *redis.Client, userID string) (*User, error) {
	// 1. Check Cache
	val, err := rdb.Get(ctx, "user:"+userID).Result()
	if err == nil {
		fmt.Println("Cache Hit")
		var user User
		json.Unmarshal([]byte(val), &user)
		return &user, nil
	} else if err != redis.Nil {
		return nil, err // Actual error
	}

	// 2. Cache Miss: Fetch from DB (simulation)
	fmt.Println("Cache Miss - Fetching from DB")
	user := &User{ID: userID, Name: "John Doe", Email: "john@example.com"}

	// 3. Update Cache
	data, _ := json.Marshal(user)
	err = rdb.Set(ctx, "user:"+userID, data, 10*time.Minute).Err()
	if err != nil {
		return nil, err
	}

	return user, nil
}

func main() {
	rdb := redis.NewClient(&redis.Options{
		Addr: "localhost:6379",
	})

	// First call: Cache Miss
	getUser(rdb, "123")
	
	// Second call: Cache Hit
	getUser(rdb, "123")
}
```

### Go Implementation: HTTP Cache Headers
```go
func cacheMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Cache for 1 hour, public (cacheable by CDNs)
        w.Header().Set("Cache-Control", "public, max-age=3600")
        next.ServeHTTP(w, r)
    })
}
```

## Interview Questions

**Q: What is the Thundering Herd problem and how do you solve it?**
**A:** It happens when a popular cache item expires, and many requests simultaneously query the database to rebuild the cache, crashing the DB.
*   *Solutions*:
    *   **Mutex/Locking**: Allow only one process to rebuild the cache.
    *   **Probabilistic Early Expiration**: Rebuild cache before it strictly expires.
    *   **Stale-While-Revalidate**: Serve stale data while updating in background.

**Q: Explain ETag vs Last-Modified.**
**A:** `ETag` is a content identifier (hash), while `Last-Modified` is a timestamp. ETags are more precise because a file might be "touched" (timestamp updated) without content changing. ETags handle sub-second updates better.

**Q: When should you use Redis vs In-Memory (process) cache?**
**A:** Use **In-Memory** for static, small configuration data where consistency across nodes isn't critical or for single-instance apps. Use **Redis** when you have multiple application instances that need to share state, when the dataset is larger than local RAM, or when you need persistence.
