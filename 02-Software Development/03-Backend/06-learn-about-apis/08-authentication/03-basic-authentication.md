---
---

## Summary
Basic Authentication is a simple authentication scheme built into the HTTP protocol. It sends the username and password as a Base64-encoded string in the `Authorization` header.

## Detailed Explanation
The format of the header is: `Authorization: Basic <base64-encoded-credentials>`. The credentials string is `username:password`.

### How it works
1. Client requests a protected resource.
2. Server responds with `401 Unauthorized` and a `WWW-Authenticate` header.
3. Client prompts the user or sends the header directly.
4. Server decodes the Base64 string and validates the credentials.

### Pros and Cons
- **Pros**: Extremely easy to implement; supported by every browser and HTTP client.
- **Cons**: Completely insecure unless used over HTTPS (credentials are sent in plain text once decoded); no built-in support for sessions or multi-factor authentication.

## Go Context
Go's standard library provides helpers for Basic Auth in the `net/http` package.

### Example: Handling Basic Auth in a Go server
```go
package main

import (
	"fmt"
	"net/http"
)

func protectedHandler(w http.ResponseWriter, r *http.Request) {
	username, password, ok := r.BasicAuth()
	if !ok || username != "admin" || password != "password123" {
		w.Header().Set("WWW-Authenticate", `Basic realm="Restricted"`)
		http.Error(w, "Unauthorized", http.StatusUnauthorized)
		return
	}
	fmt.Fprintf(w, "Welcome to the secret area, %s!", username)
}

func main() {
	http.HandleFunc("/admin", protectedHandler)
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions
- **Q: Is Base64 encoding a form of encryption?**
- **A:** No. Base64 is an encoding scheme, not encryption. Anyone who sees the string can easily decode it to retrieve the original username and password. This is why Basic Auth *must* be used over HTTPS.

- **Q: What is the `WWW-Authenticate` header?**
- **A:** It's a response header sent by the server with a `401 Unauthorized` status code. It tells the client which authentication method is required (e.g., `Basic` or `Bearer`).

- **Q: When would you use Basic Auth today?**
- **A:** It's still common for simple internal tools, local development, or legacy systems where more complex flows like OAuth or JWT are unnecessary.
