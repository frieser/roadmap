---
---

## Summary
HTTP (HyperText Transfer Protocol) is an application-layer protocol for transmitting hypermedia documents, such as HTML. It was designed for communication between web browsers and web servers, but it can also be used for other purposes. HTTP follows a classical client-server model, with a client opening a connection to make a request, then waiting until it receives a response. It is a stateless protocol, meaning that the server does not keep any data (state) between two requests.

## Detailed Explanation

### Request and Response
An HTTP transaction consists of a request and a response.
-   **Request**: Composed of a Method (GET, POST, etc.), a URL, Headers (metadata), and an optional Body.
-   **Response**: Composed of a Status Code (e.g., 200 OK, 404 Not Found), Headers, and a Body (the requested resource).

### HTTP Methods (Verbs)
-   **GET**: Retrieve a resource.
-   **POST**: Create a new resource.
-   **PUT**: Update an existing resource (replace).
-   **PATCH**: Partially update a resource.
-   **DELETE**: Remove a resource.

### Status Codes
-   **1xx**: Informational
-   **2xx**: Success (e.g., 200 OK, 201 Created)
-   **3xx**: Redirection (e.g., 301 Moved Permanently)
-   **4xx**: Client Error (e.g., 400 Bad Request, 404 Not Found, 401 Unauthorized)
-   **5xx**: Server Error (e.g., 500 Internal Server Error, 503 Service Unavailable)

### HTTP Versions
-   **HTTP/1.1**: Introduced persistent connections (Keep-Alive) and chunked transfer encoding.
-   **HTTP/2**: Introduced binary framing, multiplexing (multiple requests over one TCP connection), and header compression (HPACK).
-   **HTTP/3**: Based on QUIC (UDP), it eliminates head-of-line blocking and improves performance in lossy networks.

### HTTPS
HTTPS is HTTP over SSL/TLS. It provides encryption, data integrity, and authentication, ensuring that the communication between the client and server is secure from eavesdropping and tampering.

## Go-Specific Context/Examples

Go's standard library includes the powerful `net/http` package, which is used for both creating web servers and making HTTP requests.

### Example: Basic HTTP Server in Go
```go
package main

import (
	"fmt"
	"net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
	// r is the Request object (contains method, URL, headers)
	// w is the ResponseWriter to send data back
	fmt.Fprintf(w, "Hello, you've requested: %s\n", r.URL.Path)
}

func main() {
	http.HandleFunc("/", helloHandler)

	fmt.Println("Starting server on :8080")
	if err := http.ListenAndServe(":8080", nil); err != nil {
		fmt.Printf("Error: %v\n", err)
	}
}
```

### Example: Making an HTTP Request in Go
```go
package main

import (
	"fmt"
	"io"
	"net/http"
)

func main() {
	resp, err := http.Get("https://api.github.com")
	if err != nil {
		fmt.Printf("Request failed: %v\n", err)
		return
	}
	defer resp.Body.Close()

	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Status Code: %d\n", resp.StatusCode)
	fmt.Printf("Body length: %d\n", len(body))
}
```

### Go Application
-   **DefaultServeMux**: The standard router in `net/http`.
-   **Middleware**: Go's functional nature makes it easy to wrap handlers for logging, authentication, etc.
-   **Performance**: The `net/http` server is production-ready and highly performant, capable of handling many concurrent requests out of the box.

## Interview Questions

**Q: What does it mean that HTTP is "stateless"?**
**A:** It means the server doesn't store any information about the client between requests. Each request is independent. To maintain state (like a logged-in session), developers use cookies or tokens (JWT) passed in headers.

**Q: Explain the difference between PUT and PATCH.**
**A:** PUT is used to replace the entire resource with the provided payload. PATCH is used for partial updates, where only the fields that need to change are sent.

**Q: How did HTTP/2 improve performance over HTTP/1.1?**
**A:** HTTP/2 introduced **multiplexing**, allowing multiple requests and responses to be sent simultaneously over a single TCP connection, reducing latency and overcoming the limit of concurrent connections in HTTP/1.1. It also used binary framing and header compression.
