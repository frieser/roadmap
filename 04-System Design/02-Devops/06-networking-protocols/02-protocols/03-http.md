---
---

# HTTP (Hypertext Transfer Protocol)

HTTP is the foundation of data communication for the World Wide Web and the standard protocol for RESTful APIs. For DevOps engineers, understanding HTTP is critical for debugging services, configuring load balancers, and optimizing API performance.

## Summary

HTTP is a stateless, request-response protocol operating at **Layer 7** of the OSI model. A client (browser/script) sends a **Request** (Method, URL, Headers, Body) to a server, which returns a **Response** (Status Code, Headers, Body). It typically runs over TCP port 80.

## Detailed Explanation

### 1. HTTP Methods (Verbs)
*   **GET**: Retrieve a resource. Should be safe and idempotent.
*   **POST**: Create a new resource. Not idempotent.
*   **PUT**: Update/Replace a resource entirely. Idempotent.
*   **PATCH**: Partial update of a resource.
*   **DELETE**: Remove a resource.

### 2. Status Codes
*   **2xx (Success)**: 200 OK, 201 Created.
*   **3xx (Redirection)**: 301 Moved Permanently, 302 Found (Temporary).
*   **4xx (Client Error)**: 400 Bad Request, 401 Unauthorized (Authentication), 403 Forbidden (Authorization), 404 Not Found.
*   **5xx (Server Error)**: 500 Internal Server Error, 502 Bad Gateway (Upstream error), 503 Service Unavailable.

### 3. Key Headers for DevOps
*   `Host`: Essential for Virtual Hosting (routing multiple domains on one IP).
*   `User-Agent`: Client identification.
*   `Content-Type`: Media type of the body (e.g., `application/json`).
*   `Cache-Control`: Caching directives for CDNs and browsers.
*   `X-Forwarded-For`: The original IP of the client (added by Load Balancers).

---

## Go Implementation Example

Go's standard library `net/http` is production-ready and powers many high-performance web servers.

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
)

func main() {
	// --- 1. HTTP Server ---
	// HandleFunc registers a handler for the given pattern
	http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
		// Inspect method
		if r.Method != http.MethodGet {
			http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
			return
		}
		
		// Set headers
		w.Header().Set("Content-Type", "application/json")
		
		// Write response
		w.WriteHeader(http.StatusOK)
		io.WriteString(w, `{"message": "Hello, DevOps!"}`)
	})

	// Run server in a goroutine so we can run the client below
	go func() {
		fmt.Println("Server starting on :8080...")
		if err := http.ListenAndServe(":8080", nil); err != nil {
			log.Fatal(err)
		}
	}()

	// --- 2. HTTP Client ---
	// Wait a moment for server to start (in real app use proper sync)
	// Make a GET request
	resp, err := http.Get("http://localhost:8080/hello")
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	// Read body
	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Client received: %s (Status: %s)\n", body, resp.Status)
}
```

## Interview Questions

**Q: What is the difference between `PUT` and `PATCH`?**
**A:** `PUT` is used to replace a resource entirely. If you only provide one field in the payload, `PUT` should technically nullify all other fields of the resource. `PATCH` is used to apply partial modifications to a resource (e.g., updating just the email address of a user).

**Q: Explain the difference between 401 and 403.**
**A:** **401 Unauthorized** means "I don't know who you are" (Authentication failed or missing). **403 Forbidden** means "I know who you are, but you aren't allowed to do this" (Authorization/Permissions failed).

**Q: What does it mean that HTTP is "stateless"?**
**A:** It means that each request is independent; the server does not retain information about previous requests from the same client. To maintain a "session" (state), applications must use external mechanisms like Cookies or JWT tokens sent in the headers.
