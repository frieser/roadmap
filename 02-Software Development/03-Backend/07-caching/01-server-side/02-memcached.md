---
---

## Summary
Memcached is a general-purpose distributed memory-caching system. It is often used to speed up dynamic database-driven websites by caching data and objects in RAM to reduce the number of times an external data source (such as a database or API) must be read.

## Detailed Explanation
Memcached is known for its simplicity and efficiency. Unlike Redis, it is strictly a volatile, key-value store.

### Key Characteristics
- **Simple Key-Value**: Stores only strings (keys) and blobs (values).
- **Multithreaded Architecture**: Can scale vertically by using multiple CPU cores, which traditionally gave it an edge over the (originally) single-threaded Redis for very high throughput.
- **LRU Eviction**: Automatically removes the least recently used items when it runs out of space.
- **Slab Allocation**: Efficiently manages memory by pre-allocating "slabs" for different object sizes, preventing memory fragmentation.

### Use Cases
- Simple object caching.
- Session storage where persistence isn't required.
- High-concurrency environments with simple data access patterns.

## Go Context
The most popular Go client is `bradfitz/gomemcache`.

### Example: Using Memcached in Go
```go
package main

import (
	"fmt"
	"github.com/bradfitz/gomemcache/memcache"
)

func main() {
	// Connect to one or more servers
	mc := memcache.New("127.0.0.1:11211")

	// Set a value
	err := mc.Set(&memcache.Item{Key: "foo", Value: []byte("bar"), Expiration: 60})
	if err != nil {
		fmt.Println("Error setting:", err)
	}

	// Get a value
	it, err := mc.Get("foo")
	if err != nil {
		fmt.Println("Error getting:", err)
	} else {
		fmt.Printf("Key: %s, Value: %s\n", it.Key, it.Value)
	}
}
```

## Interview Questions
- **Q: Does Memcached support data persistence?**
- **A:** No. Memcached is strictly an in-memory cache. If the server restarts, all data is lost.

- **Q: How does Memcached scale?**
- **A:** It scales horizontally by adding more nodes. The client is responsible for choosing which node to store/retrieve data from, usually via consistent hashing.

- **Q: What is Slab Allocation?**
- **A:** It's a memory management technique where memory is divided into chunks of various sizes (slabs). When an object needs to be cached, it's placed in the smallest slab that can fit it. This avoids the overhead of constant memory allocation/deallocation and prevents fragmentation.
