#Golang
---
---

## Summary

Beyond cancellation, `context` allows passing request-scoped values (like User ID, Auth Token, Request ID) down the call chain without polluting function signatures. Use `context.WithValue` for this. However, use it sparingly: only for data that "transcends" the API boundaries, not for regular function parameters.

## Detailed Explanation

### Passing Values

```go
// Write
ctx := context.WithValue(parentCtx, key, value)

// Read
val := ctx.Value(key)
```

### Best Practices for Values

1.  **Use Custom Keys**: Never use `string` or basic types as keys to avoid collisions between packages. Define a private custom type.
2.  **Type Safety**: Provide helper functions (`WithUser`, `GetUser`) to handle type assertions, keeping the usage clean for callers.
3.  **Scope**: Use for data required by middleware or deep logging (Trace IDs), not for core business logic inputs (arguments).

### Example: HTTP Middleware

```go
package main

import (
    "context"
    "fmt"
    "net/http"
)

// Private key type prevents collisions
type contextKey string

const userKey contextKey = "user"

// Middleware injects value
func authMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Authenticate... found user "Alice"
        ctx := context.WithValue(r.Context(), userKey, "Alice")
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// Handler retrieves value
func handleProfile(w http.ResponseWriter, r *http.Request) {
    user, ok := r.Context().Value(userKey).(string)
    if !ok {
        http.Error(w, "Unauthorized", http.StatusUnauthorized)
        return
    }
    fmt.Fprintf(w, "Hello, %s", user)
}
```

### Common Use Cases

1.  **Request IDs**: Injecting a unique ID at the edge (Load Balancer/API Gateway) and propagating it to logs and DB queries for tracing.
2.  **Authentication**: Passing the authenticated user/token from middleware to controllers.
3.  **Telemetry**: OpenTelemetry uses context to propagate Trace and Span objects.
4.  **Database Transactions**: Some libraries store the active SQL transaction in context to allow repository methods to join the transaction implicitly.

### Anti-Pattern: Implicit API

Avoid using context values for required arguments.

```go
// Bad: Dependencies hidden in context
func Process(ctx context.Context) {
    db := ctx.Value("db").(*DB) // Panic if missing!
    // ...
}

// Good: Explicit dependencies
func Process(ctx context.Context, db *DB) {
    // ...
}
```

## Interview Questions

**Q: Why should you define custom types for context keys?**

**A:** `context.WithValue` stores data in a generic tree. If two packages both use the string `"userID"` as a key, they will overwrite or read each other's data, leading to subtle bugs. Using an unexported custom type (e.g., `type key int`) guarantees that the key is unique to your package and cannot be accessed or collided with by external code.

**Q: When is it appropriate to use `context.Value`?**

**A:** Use it for **request-scoped data** that is not essential to the function's core logic but is part of the request's metadata. Examples include Request IDs, Trace IDs, Authentication tokens, and Logger instances. Do **not** use it for optional parameters or to bypass function signatures for dependency injection.

**Q: How does `context.Value` affect performance?**

**A:** `context.WithValue` creates a linked list of context nodes. Searching for a value (`ctx.Value(key)`) involves traversing this list linearly up to the root. For deep call chains with many values, this is O(N). While generally fast enough for metadata, it is not optimized for high-performance lookups compared to a map or struct field.
