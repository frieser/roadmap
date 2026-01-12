#API #HTTP
---
---

## Summary
**HTTP Methods** (often called verbs) indicate the desired action to be performed on a specific resource. They are the primary mechanism for state transitions in RESTful APIs. The most common methods are **GET** (retrieve), **POST** (create), **PUT** (replace), **PATCH** (partial update), and **DELETE** (remove). Understanding their properties—specifically **Safety** and **Idempotency**—is crucial for reliable API design.

## Detailed Explanation

### Core Methods

| Method | Definition & Intended Use | Safe? | Idempotent? |
| :--- | :--- | :--- | :--- |
| **GET** | Retrieves a representation of the resource. Should only be used for reading data. | **Yes** | **Yes** |
| **POST** | Submits data to be processed (e.g., creating a new record). Often non-idempotent. | No | No |
| **PUT** | Replaces the target resource with the request payload. If it doesn't exist, it can create it. | No | **Yes** |
| **PATCH** | Applies partial modifications to a resource. Only updates the fields provided. | No | No* |
| **DELETE** | Removes the specified resource. | No | **Yes** |
| **HEAD** | Same as GET but returns no body (headers only). Useful for checking existence or metadata. | **Yes** | **Yes** |
| **OPTIONS** | Returns the communication options (e.g., allowed methods, CORS) for the target resource. | **Yes** | **Yes** |

*> Note: PATCH is not idempotent by definition (RFC 5789), though it can be implemented that way.*

### Key Concepts
- **Safe**: The operation does not modify the server state (read-only).
- **Idempotent**: Making the same request multiple times has the same effect as making it once. (e.g., `DELETE /users/1` is idempotent; the first time it deletes, the second time it does nothing or returns 404, but the server state (user gone) is the same).

---

## Go Application

In Go, handling different HTTP methods is done within the `http.Handler`.

### 1. Traditional Switch (All Versions)
Before Go 1.22, you typically used a switch statement on `r.Method`.

```go
func userHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        // Handle GET: Retrieve user
        fmt.Fprint(w, "User details")
    case http.MethodPost:
        // Handle POST: Create user
        w.WriteHeader(http.StatusCreated)
        fmt.Fprint(w, "User created")
    default:
        // Handle unsupported methods
        w.Header().Set("Allow", "GET, POST")
        http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
    }
}
```

### 2. Method-Based Routing (Go 1.22+)
Go 1.22 introduced enhanced routing patterns in `http.ServeMux`, allowing you to specify the method directly in the pattern.

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	mux := http.NewServeMux()

	// Pattern: "METHOD /path"
	mux.HandleFunc("GET /users/{id}", func(w http.ResponseWriter, r *http.Request) {
		id := r.PathValue("id")
		fmt.Fprintf(w, "Fetching user %s", id)
	})

	mux.HandleFunc("POST /users", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprint(w, "Creating new user")
	})

	mux.HandleFunc("DELETE /users/{id}", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprint(w, "Deleting user")
	})

	http.ListenAndServe(":8080", mux)
}
```

---

## Interview Questions

**Q: What is the difference between PUT and PATCH?**
**A:** `PUT` is a **replacement** operation; the client sends the *complete* new state of the resource. If fields are missing, they should be removed or set to null. `PATCH` is a **partial update**; the client sends only the fields that need to change. `PUT` is idempotent, while `PATCH` is generally not.

**Q: Why is DELETE considered idempotent even if subsequent calls return 404?**
**A:** Idempotency relates to the **server state**, not the response code. After the first DELETE call, the resource is gone. Subsequent DELETE calls do not change this state (the resource remains gone), so the side effect is the same.

**Q: When would you use HEAD instead of GET?**
**A:** Use `HEAD` when you want to check if a resource exists, valid modification dates (`Last-Modified`), or the size of the content (`Content-Length`) without downloading the entire body. This saves bandwidth.

**Q: What status code should be returned if the method is not supported?**
**A:** Return **405 Method Not Allowed**. You should also include an `Allow` header listing the supported methods (e.g., `Allow: GET, POST`).
