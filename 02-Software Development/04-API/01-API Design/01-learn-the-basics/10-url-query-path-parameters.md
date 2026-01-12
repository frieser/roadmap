#API
---
---

## Summary
In RESTful API design, URL parameters are used to pass data to the server. **Path Parameters** are part of the URL path and are used to uniquely identify a specific resource (e.g., `/users/123`). **Query Parameters** appear after a question mark (`?`) in the URL and are used to filter, sort, or paginate a collection of resources (e.g., `/users?sort=desc`). Choosing the right parameter type ensures an intuitive and standard-compliant API.

## Detailed Explanation

### 1. Path Parameters (Identifying Resources)
Path parameters are variable parts of a URL path. They are used to point to a specific resource within a collection or to define hierarchical relationships.

*   **Syntax**: `/resource/{id}`
*   **Purpose**: Identification. They are mandatory because the URL would point to a different resource or collection without them.
*   **Example**: In `/books/978-3-16-148410-0`, `978-3-16-148410-0` is a path parameter identifying a specific book by its ISBN.

### 2. Query Parameters (Sorting, Filtering, Pagination)
Query parameters are appended to the end of the URL. They are key-value pairs used to modify the behavior of the request or the representation of the resource collection.

*   **Syntax**: `/resource?key1=value1&key2=value2`
*   **Purpose**: Filtering, Sorting, Searching, and Pagination. They are typically optional.
*   **Examples**:
    *   **Filtering**: `/products?category=electronics`
    *   **Sorting**: `/users?sort=last_name&order=asc`
    *   **Pagination**: `/orders?page=2&limit=50`
    *   **Searching**: `/articles?q=golang`

### 3. When to use which? Best practices (REST)
Following RESTful conventions helps make your API predictable:

| Feature | Path Parameters | Query Parameters |
| :--- | :--- | :--- |
| **Primary Role** | Identifying a resource | Filtering/Modifying the result |
| **Requirement** | Mandatory | Usually Optional |
| **Visibility** | Part of the resource hierarchy | Search/State metadata |
| **REST Archetype** | Document/Singleton identification | Collection manipulation |

**Best Practices**:
*   **Use Nouns**: URIs should represent resources (nouns), not actions (verbs).
*   **Hierarchy**: Use path parameters to indicate "Parent-Child" relationships (e.g., `/users/{id}/orders`).
*   **Avoid Overcrowding**: Don't put too many path parameters in a single URI. If it's more than 2-3 levels deep, reconsider the resource design.
*   **Clean URLs**: Prefer query parameters for any optional data.

### 4. Go Implementation: gorilla/mux vs net/http

In Go, handling these parameters depends on the router being used.

#### Using `net/http` (Standard Library)
The standard library provides easy access to query parameters, but path parameter parsing is manual (unless using Go 1.22+ enhanced routing).

```go
package main

import (
	"fmt"
	"net/http"
)

func UserHandler(w http.ResponseWriter, r *http.Request) {
	// Query Parameters
	query := r.URL.Query()
	sort := query.Get("sort") // Returns "" if not present
	
	// Path Parameters (Manual parsing in older net/http)
	// Example path: /users/123
	// path := r.URL.Path
	
	fmt.Fprintf(w, "Sorting by: %s\n", sort)
}
```

#### Using `gorilla/mux`
`gorilla/mux` is a popular router that simplifies path variable extraction.

```go
package main

import (
	"fmt"
	"net/http"
	"github.com/gorilla/mux"
)

func ArticleHandler(w http.ResponseWriter, r *http.Request) {
	// Path Parameters
	vars := mux.Vars(r)
	id := vars["id"]
	
	// Query Parameters (same as net/http)
	category := r.URL.Query().Get("category")
	
	fmt.Fprintf(w, "Article ID: %s, Category Filter: %s", id, category)
}

func main() {
	r := mux.NewRouter()
	r.HandleFunc("/articles/{id}", ArticleHandler)
	http.ListenAndServe(":8080", r)
}
```

## Interview Questions

**Q: What is the main difference between a path parameter and a query parameter?**
**A:** Path parameters are used to identify a specific resource and are part of the URL path itself (e.g., `/users/5`). Query parameters are used to filter, sort, or paginate a collection and appear after the `?` (e.g., `/users?role=admin`).

**Q: When should you use a query parameter instead of a path parameter?**
**A:** Use query parameters for optional values, filtering, sorting, or when the value doesn't change the identity of the resource being accessed. Use path parameters for mandatory values that are essential to identifying the resource.

**Q: How do you handle multiple query parameters in a URL?**
**A:** Multiple query parameters are separated by the ampersand (`&`) symbol. For example: `/products?category=shoes&color=blue&size=42`.

**Q: How does `gorilla/mux` help with path parameters compared to the standard `net/http` (pre-1.22)?**
**A:** `gorilla/mux` allows defining named variables in the route pattern (e.g., `/users/{id}`) and provides the `mux.Vars(r)` function to easily extract them into a map. Standard `net/http` (pre-1.22) required manual string splitting or regex to extract variables from the path string.
