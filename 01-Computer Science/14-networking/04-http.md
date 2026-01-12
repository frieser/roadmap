---
---

## Summary
**HTTP (Hypertext Transfer Protocol)** is the foundation of data communication for the World Wide Web. It is an **Application Layer** protocol (Layer 7) that follows a **client-server** model. In DevOps, HTTP is the primary protocol for REST APIs, health checks, and service-to-service communication.

## Core Concepts

### HTTP Methods (Verbs)
| Method | Purpose | Idempotent |
| :--- | :--- | :--- |
| **GET** | Retrieve a resource. | Yes |
| **POST** | Create a new resource or perform an action. | No |
| **PUT** | Replace a resource or create it if it doesn't exist. | Yes |
| **PATCH** | Partially update a resource. | No |
| **DELETE** | Remove a resource. | Yes |
| **HEAD** | Same as GET but returns only headers (useful for health checks). | Yes |
| **OPTIONS** | Describe communication options for the target resource (CORS). | Yes |

### Common Headers
- **Host**: Domain name of the server (required in HTTP/1.1).
- **User-Agent**: Information about the client software.
- **Content-Type**: Media type of the body (e.g., `application/json`).
- **Authorization**: Credentials for authenticating the client (e.g., `Bearer <token>`).
- **Accept**: Media types the client can handle.
- **Cache-Control**: Directives for caching mechanisms.

### Status Codes
- **1xx (Informational)**: Request received, continuing process.
- **2xx (Success)**: Action received, understood, and accepted (e.g., `200 OK`, `201 Created`).
- **3xx (Redirection)**: Further action needs to be taken (e.g., `301 Moved Permanently`, `302 Found`).
- **4xx (Client Error)**: Request contains bad syntax or cannot be fulfilled (e.g., `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`).
- **5xx (Server Error)**: Server failed to fulfill an apparently valid request (e.g., `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`).

## Go Implementation: Basic Server and Client

### HTTP Server
```go
package main

import (
	"fmt"
	"net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, you've requested: %s\n", r.URL.Path)
}

func main() {
	http.HandleFunc("/", helloHandler)
	fmt.Println("Server starting on :8080")
	if err := http.ListenAndServe(":8080", nil); err != nil {
		panic(err)
	}
}
```

### HTTP Client
```go
package main

import (
	"fmt"
	"io"
	"net/http"
)

func main() {
	resp, err := http.Get("http://localhost:8080/")
	if err != nil {
		fmt.Printf("Error: %v\n", err)
		return
	}
	defer resp.Body.Close()

	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Status: %s\nHeaders: %v\nBody: %s\n", resp.Status, resp.Header, string(body))
}
```

## Interview Questions
- **Q: What is the difference between HTTP/1.1 and HTTP/2?**
  - **A:** HTTP/1.1 is text-based and suffers from Head-of-Line blocking. HTTP/2 is binary, supports multiplexing (multiple requests over one TCP connection), header compression (HPACK), and server push.
- **Q: What is an idempotent method?**
  - **A:** A method is idempotent if performing the same request multiple times has the same effect as performing it once (e.g., GET, PUT, DELETE). POST is not idempotent.
- **Q: How do status codes 502 and 504 differ?**
  - **A:** `502 Bad Gateway` means the proxy received an invalid response from the upstream server. `504 Gateway Timeout` means the proxy didn't receive a response from the upstream server within the timeout period.
