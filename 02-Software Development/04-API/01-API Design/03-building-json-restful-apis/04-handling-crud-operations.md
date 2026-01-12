#API
---
---

## Summary
CRUD (Create, Read, Update, Delete) operations form the backbone of most RESTful services. In a REST architecture, these operations are mapped to standard HTTP verbs, ensuring a predictable and uniform interface for managing resources. This note covers the mapping of verbs to operations, appropriate status codes, and a practical implementation in Go.

## Detailed Development

### 1. Mapping Verbs to Operations
RESTful APIs leverage HTTP methods to express semantic actions on resources.

| Operation | HTTP Verb | Description | Success Code | Idempotent |
| :--- | :--- | :--- | :--- | :--- |
| **Create** | `POST` | Creates a new resource. | `201 Created` | No |
| **Read** | `GET` | Retrieves a resource or a list of resources. | `200 OK` | Yes |
| **Update** | `PUT` | Replaces an existing resource entirely. | `200 OK` / `204` | Yes |
| **Partial Update**| `PATCH`| Modifies specific fields of a resource. | `200 OK` | No (usually) |
| **Delete** | `DELETE` | Removes a resource. | `204 No Content` | Yes |

### 2. Response Codes for CRUD
Selecting the correct HTTP status code is crucial for API clarity:
*   **200 OK**: Standard response for successful GET, PUT, or PATCH.
*   **201 Created**: Specifically for POST when a resource is successfully created. Usually includes a `Location` header.
*   **204 No Content**: Successful request that returns no body (standard for DELETE and sometimes PUT).
*   **400 Bad Request**: The request body or parameters are invalid.
*   **404 Not Found**: The target resource does not exist.

### 3. Partial Updates (PATCH)
While `PUT` is intended for full resource replacement (if you omit a field, it might be nullified), `PATCH` is used for partial modifications.
*   **Implementation Strategy**: In Go, handling partial updates often involves using pointers in structs or decoding into a `map[string]interface{}` to distinguish between a field being "missing" vs "explicitly set to zero/null".

### 4. Go: Complete CRUD Handler Example
This example demonstrates a thread-safe CRUD implementation using standard library `net/http` and a simple in-memory map.

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"sync"
)

type Item struct {
	ID    string `json:"id"`
	Name  string `json:"name"`
	Price float64 `json:"price"`
}

var (
	store = make(map[string]Item)
	mu    sync.RWMutex
)

func itemHandler(w http.ResponseWriter, r *http.Request) {
	id := r.URL.Path[len("/items/"):]

	switch r.Method {
	case http.MethodGet:
		mu.RLock()
		item, ok := store[id]
		mu.RUnlock()
		if !ok {
			http.Error(w, "Item not found", http.StatusNotFound)
			return
		}
		json.NewEncoder(w).Encode(item)

	case http.MethodPost:
		var item Item
		if err := json.NewDecoder(r.Body).Decode(&item); err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
			return
		}
		mu.Lock()
		store[item.ID] = item
		mu.Unlock()
		w.WriteHeader(http.StatusCreated)
		json.NewEncoder(w).Encode(item)

	case http.MethodPut:
		var item Item
		if err := json.NewDecoder(r.Body).Decode(&item); err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
			return
		}
		mu.Lock()
		if _, ok := store[id]; !ok {
			mu.Unlock()
			http.Error(w, "Item not found", http.StatusNotFound)
			return
		}
		store[id] = item
		mu.Unlock()
		w.WriteHeader(http.StatusNoContent)

	case http.MethodDelete:
		mu.Lock()
		delete(store, id)
		mu.Unlock()
		w.WriteHeader(http.StatusNoContent)

	default:
		w.Header().Set("Allow", "GET, POST, PUT, DELETE")
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
	}
}

func main() {
	http.HandleFunc("/items/", itemHandler)
	fmt.Println("Server starting on :8080...")
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions
*   **Q: What is the difference between PUT and PATCH?**
    *   **A:** `PUT` is used for full replacement of a resource (idempotent), while `PATCH` is used for partial updates (not necessarily idempotent).
*   **Q: Why is POST not idempotent?**
    *   **A:** Multiple identical `POST` requests will typically result in multiple resources being created (e.g., placing the same order twice), whereas multiple `PUT` requests will result in the same state.
*   **Q: When should you return a 204 No Content?**
    *   **A:** When an operation (like `DELETE` or `PUT`) is successful, but there is no need to return a response body to the client.
*   **Q: How do you handle "null" vs "omitted" fields in a Go PATCH request?**
    *   **A:** Use pointers in the struct fields (e.g., `*string`). If the pointer is `nil`, the field was omitted. If it points to an empty string, it was explicitly set to empty.
