---
---

# Hash Table

## Abstract
A **Hash Table** (or Hash Map) is a data structure that implements an associative array abstract data type, a structure that can map keys to values. It uses a **hash function** to compute an index into an array of buckets or slots, from which the desired value can be found. In Go, this is implemented as the built-in `map` type. It offers **O(1)** average time complexity for lookups, insertions, and deletions, making it one of the most important and frequently used data structures in software engineering.

## Development

### Core Concept
The core idea is to transform a key (e.g., a string "user_123") into an integer index (e.g., `42`) using a **hash function**. This index points to a specific location in memory where the value is stored.

1.  **Hash Function**: A function that takes an input (key) and returns a fixed-size string of bytes (hash value). It should be deterministic and distribute keys uniformly.
2.  **Buckets**: The internal storage array. The hash determines which bucket a key belongs to.
3.  **Collision Handling**: When two keys hash to the same bucket (a collision), the hash table must handle it. Common strategies:
    *   **Chaining**: Each bucket points to a linked list (or another data structure) of entries.
    *   **Open Addressing**: If a bucket is full, find the next available slot using a probing sequence (linear, quadratic, etc.).

### Go Implementation Details
Go's `map` is a sophisticated hash table implementation.
- **Structure**: It is backed by a pointer to a `hmap` struct in the runtime.
- **Buckets**: Data is stored in an array of buckets. Each bucket holds up to **8 key/value pairs**.
- **Collision**: Go uses a form of **chaining**. If a bucket overflows (more than 8 keys), it links to an **overflow bucket**.
- **Hashing**: Go uses distinct hash functions per architecture/type (e.g., AES-based hashing on hardware that supports it) to prevent HashDoS attacks.

### Time Complexity

| Operation | Average Case | Worst Case | Description |
|-----------|--------------|------------|-------------|
| **Access**| **O(1)**     | O(n)       | Constant time mostly. Linear if many collisions (bad hash/DoS). |
| **Search**| **O(1)**     | O(n)       | Same as access. |
| **Insert**| **O(1)**     | O(n)       | May trigger resizing (evacuation) if load factor > 6.5. |
| **Delete**| **O(1)**     | O(n)       | Simple unlinking/marking. |

## Code Examples (Go)

Go provides the `map` keyword as a built-in type.

### 1. Basic Usage
```go
package main

import "fmt"

func main() {
    // Initialization: make(map[KeyType]ValueType)
    // Always use make() to avoid nil map panic on assignment
    scores := make(map[string]int)

    // Insertion
    scores["Alice"] = 95
    scores["Bob"] = 82

    // Access
    fmt.Println("Alice:", scores["Alice"]) // 95

    // Delete
    delete(scores, "Bob")

    // Accessing missing key returns zero value (0 for int)
    fmt.Println("Bob:", scores["Bob"]) // 0
}
```

### 2. The "Comma OK" Idiom
Since accessing a missing key returns the zero value, you need a way to distinguish between "value is 0" and "key not found".

```go
func check(m map[string]int, key string) {
    val, ok := m[key]
    if ok {
        fmt.Printf("Key '%s' exists with value %d\n", key, val)
    } else {
        fmt.Printf("Key '%s' does not exist\n", key)
    }
}
```

### 3. Iteration
Map iteration order in Go is **randomized** intentionally to prevent developers from relying on implementation details (bucket order).

```go
    colors := map[string]string{
        "red":   "#ff0000",
        "green": "#00ff00",
        "blue":  "#0000ff",
    }

    // Order will vary between runs!
    for key, val := range colors {
        fmt.Printf("%s -> %s\n", key, val)
    }
```

## Go Application & Ecosystem

### Concurrency Safety
Standard Go maps are **NOT thread-safe**. Concurrent reads are safe, but **concurrent read and write** (or write and write) will cause a fatal runtime panic (`fatal error: concurrent map read and map write`).

- **Solution 1 (Mutex)**: Use `sync.RWMutex` to guard access.
- **Solution 2 (sync.Map)**: Specialized map for specific cases (append-only caches or disjoint key sets).

### Map vs Struct vs Slice
- Use **Map** when you have dynamic keys or sparse data.
- Use **Struct** when fields are fixed and known at compile time (faster, type-safe).
- Use **Slice** for ordered lists or when doing simple sequential scans on small datasets.

### Memory Overhead
A Go map has significant overhead compared to a slice or array due to the bucket structure, pointers, and metadata. For very small collections (e.g., < 10 items), a simple slice linear search might be faster and lighter.

## Interview Preparation

### Common Questions

1.  **How does Go handle map collisions?**
    -   **Answer**: Go uses a bucket-based approach with chaining via overflow buckets. Each bucket holds 8 keys. If it fills up, it links to a new overflow bucket. It does NOT use open addressing or simple linked-list chaining per key.

2.  **What happens when a Go map grows?**
    -   **Answer**: When the load factor exceeds ~6.5, Go allocates a new, larger bucket array (usually double size) and incrementally "evacuates" (moves) keys from old buckets to new ones during insertions/deletes to avoid a "stop-the-world" pause.

3.  **Why is map iteration order random in Go?**
    -   **Answer**: To prevent reliance on the internal bucket order, which changes as the map grows. It also adds a randomization factor (seed) at the start of iteration to ensure this behavior is consistent even if the map doesn't change.

4.  **Can you take the address of a map value?**
    -   **Answer**: No. `&m["key"]` is invalid.
    -   **Reason**: Map growth might move the value to a different memory address (new bucket), invalidating the pointer.

5.  **Is `sync.Map` always better for concurrency?**
    -   **Answer**: No. `sync.Map` is optimized for two specific cases: (1) entries are only written once but read many times (cache), or (2) multiple goroutines read/write disjoint sets of keys. For standard read/write contention, a `map` with a `Mutex` is often faster and type-safe.
