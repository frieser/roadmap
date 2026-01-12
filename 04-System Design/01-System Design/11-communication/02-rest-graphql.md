---
---

## Summary
**REST (Representational State Transfer)** and **GraphQL** are the two most popular architectural styles for web APIs. REST is resource-centric, using standard HTTP methods and multiple endpoints. GraphQL is query-centric, allowing clients to request exactly the data they need from a single endpoint, solving common issues like over-fetching and under-fetching.

## Detailed Explanation

### 1. REST (Representational State Transfer)
REST treats everything as a **resource**, identified by a URI.
*   **Stateless**: Each request must contain all information needed to process it.
*   **Standard Methods**: Uses GET, POST, PUT, DELETE, PATCH.
*   **Pros**: Highly cacheable, standard-based, easy to understand, decoupled client/server.
*   **Cons**: **Over-fetching** (getting more data than needed) or **Under-fetching** (requiring multiple calls to get related data - the N+1 problem).

### 2. GraphQL
Developed by Facebook, GraphQL is a query language for APIs and a runtime for fulfilling those queries.
*   **Single Endpoint**: Typically `/graphql`.
*   **Strongly Typed Schema**: Defined using SDL (Schema Definition Language).
*   **Declarative Data Fetching**: The client specifies the shape of the response.
*   **Pros**: No over/under-fetching, strong typing, introspection (built-in documentation).
*   **Cons**: Complex caching (queries are usually POST), potential for expensive queries (DoS risk), steeper learning curve.

### Go Context: `net/http` and `gqlgenc`
In Go, REST is typically handled by `net/http` or frameworks like Gin/Echo. For GraphQL, `gqlgen` is the standard for servers, and `gqlgenc` is popular for type-safe clients.

#### REST Endpoint in Go
```go
package main

import (
	"encoding/json"
	"net/http"
)

type User struct {
	ID   int    `json:"id"`
	Name string `json:"name"`
}

func userHandler(w http.ResponseWriter, r *http.Request) {
	user := User{ID: 1, Name: "Alice"}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(user)
}

func main() {
	http.HandleFunc("/user", userHandler)
	http.ListenAndServe(":8080", nil)
}
```

#### GraphQL Client with `gqlgenc`
`gqlgenc` generates type-safe Go code from your GraphQL queries.
```go
// After running gqlgenc to generate client.go
package main

import (
	"context"
	"fmt"
	"github.com/gqlgo/gqlgenc/clientv2"
	"myproject/generated"
)

func main() {
	c := generated.NewClient(http.DefaultClient, "https://api.annict.com/graphql")
	ctx := context.Background()
	
	// Type-safe query execution
	res, err := c.GetUser(ctx, "username")
	if err != nil {
		return
	}
	fmt.Println(res.User.Name)
}
```

## Interview Questions
*   **Q: What is over-fetching and how does GraphQL solve it?**
    *   **A:** Over-fetching occurs when an API returns more data fields than the client needs (e.g., returning a full user profile when only the username is needed). GraphQL solves this by allowing the client to specify exactly which fields it wants in the query.
*   **Q: Why is caching harder in GraphQL compared to REST?**
    *   **A:** REST uses standard HTTP GET requests with unique URLs for resources, which browsers and CDNs can easily cache. GraphQL usually uses POST for all queries, and the same endpoint returns different data based on the body, making traditional URL-based caching impossible.
*   **Q: When would you choose REST over GraphQL?**
    *   **A:** Choose REST when you need simple, highly cacheable resources, when your consumers are external and want a standard interface, or when your data structure is very flat and doesn't involve complex relationships.
