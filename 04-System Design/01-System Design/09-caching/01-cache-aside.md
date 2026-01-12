---
---

# Cache-Aside (Lazy Loading)

**Cache-Aside** is the most common caching strategy. In this pattern, the application is responsible for managing both the cache and the database. The cache does not communicate directly with the database.

## **How it Works**
1. **Read Request**:
   - The application checks the cache for the requested data.
   - **Cache Hit**: Data is returned immediately.
   - **Cache Miss**: The application fetches data from the database, stores it in the cache, and then returns it to the user.
2. **Write Request**:
   - Data is written directly to the database.
   - The corresponding cache entry is typically invalidated (deleted) to ensure consistency on the next read.

## **Pros**
- **Efficient Storage**: Only data that is actually requested is cached (Lazy Loading).
- **Resilience**: If the cache fails, the application can still function by falling back to the database.
- **Flexible Data Model**: The cache can store data in a different format than the database (e.g., objects vs. rows).

## **Cons**
- **Cache Miss Penalty**: The first request for any piece of data always incurs a "double penalty" (cache miss + DB fetch + cache write).
- **Stale Data**: If data is updated in the database without invalidating the cache, the application might serve old data until the TTL expires.

## **Go Context: Implementation**

### **Standard Map + RWMutex**
For a simple in-memory cache-aside implementation:

```go
type Service struct {
    db    *sql.DB
    cache map[string]User
    mu    sync.RWMutex
}

func (s *Service) GetUser(id string) (User, error) {
    // 1. Check cache (Read)
    s.mu.RLock()
    user, hit := s.cache[id]
    s.mu.RUnlock()
    if hit {
        return user, nil
    }

    // 2. Cache Miss: Fetch from DB
    err := s.db.QueryRow("SELECT ...", id).Scan(&user)
    if err != nil {
        return User{}, err
    }

    // 3. Update Cache
    s.mu.Lock()
    s.cache[id] = user
    s.mu.Unlock()

    return user, nil
}
```

### **Using Redis**
In distributed systems, Redis is typically used as the cache provider.

```go
func (s *Service) GetUser(ctx context.Context, id string) (User, error) {
    // 1. Try Redis
    val, err := s.rdb.Get(ctx, "user:"+id).Result()
    if err == nil {
        var user User
        json.Unmarshal([]byte(val), &user)
        return user, nil
    }

    // 2. Fallback to DB
    user, err := s.fetchFromDB(id)
    if err != nil {
        return User{}, err
    }

    // 3. Set in Redis with TTL
    data, _ := json.Marshal(user)
    s.rdb.Set(ctx, "user:"+id, data, 10*time.Minute)

    return user, nil
}
```

## **Best Practices**
- Always set a **TTL (Time to Live)** to prevent permanent staleness.
- Invalidate the cache entry on every database update.
- Use a library like `singleflight` in Go to prevent **Cache Stampede** (multiple concurrent requests for the same missing key hitting the DB at once).
