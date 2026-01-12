---
---

# Application Caching

**Application Caching** is implemented directly within the application code. This is where developers have the most control.

## **Types**
1. **In-Memory (Local)**:
    - Stores data in the application's RAM.
    - **Go Tools**: `sync.Map`, `map + RWMutex`, `lru-cache` libraries.
    - **Fastest** possible caching but data is lost on restart and not shared across instances.
2. **Distributed**:
    - Shared cache across multiple application instances.
    - **Tools**: Redis, Memcached.
    - **Pros**: Consistency across instances, data persists across restarts.

## **Go Context: sync.Map vs. Redis**

| Feature | `sync.Map` | `map + RWMutex` | Redis |
|---------|------------|-----------------|-------|
| **Scope** | Single Instance | Single Instance | Distributed |
| **Speed** | Nano-seconds | Nano-seconds | Milli-seconds (Network) |
| **Concurrency** | Optimized for reads | Standard locking | High (Atomic operations) |
| **Complexity** | Low | Low | Medium |

### **When to use sync.Map?**
Use `sync.Map` when:
1. The entry for a given key is only ever written once but read many times (e.g., caches that only grow).
2. Multiple goroutines read, write, and overwrite entries for disjoint sets of keys.

### **When to use Redis?**
Use Redis when:
1. You have multiple application instances and they need to share the same cache.
2. You need complex data structures (Lists, Sets, Sorted Sets).
3. You need the cache to persist if the app crashes.
