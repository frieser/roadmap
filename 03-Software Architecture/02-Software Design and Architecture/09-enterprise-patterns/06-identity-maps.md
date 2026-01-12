---
title: Identity Map Pattern
category: Enterprise Patterns
tags: [architecture, design-patterns, orm, go]
---

# Identity Map Pattern

The **Identity Map** is an enterprise application pattern that ensures each database record is loaded only once per session by keeping every loaded object in a map.

## 1. Definition and Core Problem
It acts as a cache that maintains a mapping between database identities (primary keys) and in-memory object instances.

### The Problem: Object Uniqueness
Without an Identity Map, loading the same row multiple times results in multiple distinct objects.
- **Inconsistency**: Updating one object doesn't affect the other, leading to "stale" data.
- **Performance**: Multiple database calls for the same data waste resources.
- **Lost Updates**: Writing back changes from different instances of the same record can cause data corruption.

## 2. Implementation in ORMs
The Identity Map is typically managed by a **Unit of Work** or **Persistence Context**.

- **Data Mapper**: The mapper checks the Identity Map before querying the database.
- **Session-Scoped**: It is usually scoped to a single request or database transaction to avoid memory leaks and cross-session stale data.

## 3. Trade-offs
| Pros | Cons |
| :--- | :--- |
| Guaranteed object identity (u1 == u2) | Increased memory consumption |
| Automatic caching within a request | Complexity in managing the map lifecycle |
| Prevents redundant DB queries | Potential for stale data if external updates occur |

## 4. Go (Golang) Implementation
Go ORMs like GORM often omit this pattern to stay "simple and explicit". However, for complex business logic, it can be implemented manually:

```go
type IdentityMap struct {
    mu    sync.RWMutex
    store map[string]interface{}
}

func (m *IdentityMap) Get(id int, entityType string) interface{} {
    m.mu.RLock()
    defer m.mu.RUnlock()
    return m.store[fmt.Sprintf("%s:%d", entityType, id)]
}
```

## 5. Modern Relevance (2025/2026)
- **Legacy**: Vital for stateful, long-lived server-side sessions.
- **Modern**: In stateless REST/GraphQL APIs, its benefit is smaller but still useful for complex batch processing or deep object graphs (e.g., GraphQL resolvers fetching the same entity multiple times in one request).
- **Tooling**: Heavy usage in TypeScript (MikroORM) and Java (Hibernate/JPA).

## References
- [Martin Fowler - Identity Map](https://martinfowler.com/eaaCatalog/identityMap.html)
- [Enterprise Patterns in Go](https://github.com/tmrts/go-patterns)
