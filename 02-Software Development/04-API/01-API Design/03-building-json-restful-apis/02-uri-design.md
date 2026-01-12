# API Design: URI Design

## Summary
URI (Uniform Resource Identifier) design is a critical aspect of RESTful API development. It focuses on creating intuitive, readable, and consistent paths to resources. Proper URI design uses nouns for resource names, employs standard pluralization, maintains logical hierarchies, and follows established casing conventions to ensure the API is easy to navigate and maintain.

## Detailed Explanation

### 1. Resource Naming: Nouns vs Verbs
REST is centered around **resources**, which are objects or entities. Therefore, URIs should use **nouns** instead of verbs. The action to be performed on the resource is determined by the HTTP method (GET, POST, PUT, DELETE), not the URI itself.

| Action | Bad (Verb-based) | Good (Noun-based) |
| :--- | :--- | :--- |
| Get all users | `GET /getUsers` | `GET /users` |
| Create user | `POST /createUser` | `POST /users` |
| Delete user | `POST /deleteUser/1` | `DELETE /users/1` |

### 2. Pluralization
The industry standard is to use **plural nouns** for all resources. This maintains consistency between collection endpoints and individual resource endpoints.

- **Collection**: `/users` (List all users)
- **Single Resource**: `/users/123` (Get user with ID 123)

### 3. Hierarchy and Nesting
Use the URI structure to represent relationships between resources. A forward slash (`/`) is used to show a hierarchical relationship.

- **Example**: `/users/1/orders` (Orders belonging to user 1)
- **Best Practice**: Limit nesting to **2 or 3 levels**. Deeply nested URIs (e.g., `/authors/1/books/5/chapters/10/comments`) become brittle and hard to use. If a resource can be identified uniquely by its own ID, consider a flatter structure like `/books/5/chapters/10`.

### 4. Case Convention
Consistency in casing is essential for professional APIs.
- **kebab-case**: Recommended for URIs. It is standard for URLs and highly readable (e.g., `/user-profiles`, `/order-history`).
- **snake_case**: Frequently used, especially when mapping directly to database field names (e.g., `/user_profiles`).
- **camelCase**: Generally discouraged in URIs but common in JSON response bodies.

## Go Application: Routing Implementation

In Go, you can implement these URI patterns using the standard library or popular third-party routers.

### Standard Library (net/http)
Since **Go 1.22**, the built-in `http.ServeMux` supports HTTP methods and path parameters directly.

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	mux := http.NewServeMux()

	// Pattern: METHOD PATH/{param}
	mux.HandleFunc("GET /users/{id}", func(w http.ResponseWriter, r *http.Request) {
		id := r.PathValue("id")
		fmt.Fprintf(w, "Retrieving user with ID: %s", id)
	})

	http.ListenAndServe(":8080", mux)
}
```

### Chi (Lightweight & Idiomatic)
Chi is often preferred for REST APIs because it stays close to the standard library while adding powerful features like sub-routing.

```go
package main

import (
	"net/http"
	"github.com/go-chi/chi/v5"
)

func main() {
	r := chi.NewRouter()

	r.Route("/users", func(r chi.Router) {
		r.Get("/", listUsers)          // GET /users
		r.Post("/", createUser)        // POST /users
		r.Route("/{id}", func(r chi.Router) {
			r.Get("/", getUser)        // GET /users/123
			r.Get("/orders", getOrders) // GET /users/123/orders
		})
	})

	http.ListenAndServe(":8080", r)
}
```

### Gin & Echo (High Performance)
These frameworks provide built-in parameter binding and validation.
- **Gin**: `r.GET("/users/:id", handler)`
- **Echo**: `e.GET("/users/:id", handler)`

## Interview Questions

**Q: Why should we use nouns instead of verbs in REST URIs?**
**A:** REST is resource-oriented. The HTTP methods (GET, POST, etc.) act as the verbs. Including verbs in the URI (e.g., `/deleteUser`) makes the API less intuitive and violates the principle of a uniform interface.

**Q: How do you handle deep resource nesting in a REST API?**
**A:** Avoid deep nesting (more than 3 levels). Instead, "flatten" the API by using the unique ID of the sub-resource if possible (e.g., `/orders/5` instead of `/users/1/orders/5`) or use query parameters for filtering.

**Q: What are the routing improvements in Go 1.22?**
**A:** Go 1.22 introduced "Enhanced Routing Patterns" to `http.ServeMux`. It now supports specifying the HTTP method in the pattern (e.g., `"GET /path"`) and path parameters using curly braces (e.g., `"/users/{id}"`), which can be retrieved using `r.PathValue("id")`.

**Q: What is the benefit of kebab-case in URIs?**
**A:** Kebab-case is considered more "web-friendly" and is the standard for URLs. It is easy for humans to read and avoids issues with case sensitivity that sometimes occur with camelCase in certain server configurations.
