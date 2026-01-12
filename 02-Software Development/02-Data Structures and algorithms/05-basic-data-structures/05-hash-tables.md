---
---

# Hash Tables (Maps)

## Summary
A **Hash Table** (or Map) is a data structure that maps keys to values using a hashing function. It allows for highly efficient data retrieval, with **$O(1)$ amortized time complexity** for insertion, lookup, and deletion. Go's built-in `map` type is a sophisticated hash table implementation.

## Detailed Explanation

### 1. How it Works
1.  **Hashing**: The key is passed through a hash function to generate a numeric hash code.
2.  **Bucketing**: The hash code determines which "bucket" the key-value pair belongs to.
3.  **Collision Resolution**: If two keys land in the same bucket, Go uses a strategy called "chaining" (internally using extra buckets) to store them.

### 2. Go's `map` Implementation
*   **Dynamic**: Maps grow automatically.
*   **Unordered**: Iteration order is randomized by the runtime to prevent reliance on hash collision order.
*   **Reference Type**: Maps are pointers to an underlying `hmap` struct. Passing a map to a function allows that function to modify the original map.

## Code Examples (Go)

### 1. Basic Usage
```go
package main

import "fmt"

func main() {
    // Initialization
    // make(map[KeyType]ValueType)
    scores := make(map[string]int)
    
    // Insertion
    scores["Alice"] = 100
    scores["Bob"] = 95
    
    // Lookup
    // The "comma ok" idiom checks for existence
    score, exists := scores["Charlie"]
    if !exists {
        fmt.Println("Charlie not found")
    } else {
        fmt.Println("Charlie:", score)
    }
    
    // Deletion
    delete(scores, "Bob")
}
```

### 2. The "Concurrent Write" Crash
Go maps are **not thread-safe**.

```go
// THIS WILL PANIC
go func() { m["a"] = 1 }()
go func() { m["a"] = 2 }()
```

**Solution**: Use `sync.RWMutex` or `sync.Map`.

```go
import "sync"

var mu sync.RWMutex
var m = make(map[string]int)

func safeSet(k string, v int) {
    mu.Lock()
    m[k] = v
    mu.Unlock()
}
```

## Interview Questions

**Q: What is the time complexity of map operations in Go?**
**A:** $O(1)$ on average. In the worst case (many hash collisions), it can degrade to $O(n)$, but Go's hash function (based on AES or specialized hashing) is designed to avoid this.

**Q: Can you use a slice as a map key?**
**A:** No. Map keys must be **comparable**. Slices, maps, and functions cannot be compared using `==`, so they cannot be keys. Arrays (`[N]T`) and Structs (if their fields are comparable) *can* be keys.

**Q: Why is map iteration order random in Go?**
**A:** It is a deliberate design choice by the Go team. In early versions, the order was stable-ish but relied on implementation details. To prevent developers from writing code that accidentally depends on this order (breaking if the implementation changed), they added explicit randomization.

**Q: How does a Go map grow?**
**A:** When the "load factor" (average items per bucket) exceeds a threshold (6.5), the map allocates a new array of buckets twice as large and incrementally "evacuates" (moves) old data to the new buckets during subsequent operations.
