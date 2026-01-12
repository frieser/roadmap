---
---

# Improper Instantiation

## Summary
Creating new instances of objects that manage external resources (like database connections or HTTP clients) for every request instead of pooling or reusing them. This leads to resource exhaustion (e.g., socket leakage), increased GC pressure, and high latency.

## Detailed Development
Many libraries provide abstractions for external resources. These classes typically manage their own connection pools or internal state. Instantiating them per-request is expensive because:
- **TCP/Socket Exhaustion**: Each new HTTP client or DB driver may open new sockets.
- **Handshake Overhead**: Costs of setting up TLS/TCP connections repeatedly.
- **Memory Pressure**: Increased allocation on the heap leads to more frequent Garbage Collection (GC) cycles.

### Common Examples
1. **HTTP Clients**: Creating a `new HttpClient()` in every function call.
2. **Database Drivers**: Calling `db.Connect()` and `db.Close()` for every query.
3. **Large Buffers**: Allocating large byte slices for every incoming packet instead of using a pool.

## Go-Specific Application

### 1. `sql.DB` Pooling
In Go, `sql.Open` does **not** establish a single connection. It returns a handle to a database connection pool. You should call it **once** during application startup and share the `*sql.DB` instance.

```go
// GOOD: Initialize once and share
var db *sql.DB

func main() {
    var err error
    db, err = sql.Open("postgres", connStr)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(25)
    db.SetConnMaxLifetime(5 * time.Minute)
}
```

### 2. `http.Client` Re-use
The default `http.Client` is thread-safe and manages a connection pool internally (`http.Transport`). Creating a new client for every request bypasses this pooling.

```go
// BAD: New client per request
func Fetch(url string) {
    client := &http.Client{} // High overhead, no connection reuse
    client.Get(url)
}

// GOOD: Shared client
var httpClient = &http.Client{
    Timeout: time.Second * 10,
    Transport: &http.Transport{
        MaxIdleConns:        100,
        IdleConnTimeout:     90 * time.Second,
    },
}
```

### 3. `sync.Pool` for Temporary Objects
For objects that are expensive to allocate but are used briefly and then discarded, Go provides `sync.Pool`. This reduces GC overhead by reusing memory.

```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 1024)
    },
}

func Process() {
    buf := bufferPool.Get().([]byte)
    defer bufferPool.Put(buf)
    
    // Use buf...
}
```

## Interview Preparation

### Questions
1. **What happens if you create a new `http.Client` for every request in Go?**
   - You risk exhausting available file descriptors (sockets) because the underlying TCP connections stay in `TIME_WAIT` state even after `client.Close()`.

2. **Is `sync.Pool` a cache?**
   - No. `sync.Pool` objects can be cleared by the GC at any time. It is a mechanism for memory reuse to reduce GC pressure, not for long-term state storage.

3. **How does `sql.DB` handle concurrency?**
   - It is thread-safe and manages a pool of connections automatically. You don't need to open/close it for every transaction.
