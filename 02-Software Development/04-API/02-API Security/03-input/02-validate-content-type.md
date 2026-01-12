#API
---
---

# Validate Content-Type

## Summary
Enforcing the correct `Content-Type` header is a defense against **Content Sniffing** and **MIME Confusion** attacks. Clients (browsers or scripts) should explicitly declare the format of the data they are sending (e.g., `application/json`). If the server expects JSON but blindly processes `application/x-www-form-urlencoded` because it "guesses" the format, it opens the door to cross-site request vulnerabilities and parsing errors.

**Best Practice:**
- Whitelist allowed content types (e.g., `application/json`, `multipart/form-data`).
- Reject requests with missing or mismatching headers with `415 Unsupported Media Type`.

---

## Detailed Explanation

### Why Validate Content-Type?
1.  **Parsing Security**: Go's `json.Decoder` might behave differently if fed non-JSON data.
2.  **CSRF Mitigation**: Some CORS configurations relax restrictions for "simple" content types (like `text/plain` or `application/x-www-form-urlencoded`). Enforcing `application/json` forces a preflight OPTIONS request, adding a layer of CORS protection.
3.  **Ambiguity**: Without a header, servers might try to "sniff" the content. If a user uploads a file named `avatar.jpg` that contains HTML/JS, a browser might execute it as a script (XSS).

### Go Middleware Implementation
This middleware inspects the request header before passing it to the handler.

```go
package main

import (
	"fmt"
	"mime"
	"net/http"
	"strings"
)

// EnforceJSONContentType Middleware
func EnforceJSONContentType(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// 1. Skip validation for methods without a body (GET, DELETE, HEAD)
		if r.Method == http.MethodGet || r.Method == http.MethodHead || r.Method == http.MethodDelete {
			next.ServeHTTP(w, r)
			return
		}

		// 2. Get the Content-Type header
		contentType := r.Header.Get("Content-Type")

		// 3. Parse media type (handles "application/json; charset=utf-8")
		mediaType, _, err := mime.ParseMediaType(contentType)
		if err != nil {
			http.Error(w, "Malformed Content-Type header", http.StatusBadRequest)
			return
		}

		// 4. Validate against allowed types
		if mediaType != "application/json" {
			msg := fmt.Sprintf("Unsupported Media Type: %s. Expected: application/json", mediaType)
			http.Error(w, msg, http.StatusUnsupportedMediaType)
			return
		}

		next.ServeHTTP(w, r)
	})
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/api/data", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte(`{"status": "ok"}`))
	})

	// Wrap middleware
	http.ListenAndServe(":8080", EnforceJSONContentType(mux))
}
```

---

## Interview Questions

### 1. What status code should you return if the Content-Type is invalid?
**415 Unsupported Media Type**. This is the semantic HTTP code specifically designed for this scenario, rather than a generic `400 Bad Request`.

### 2. Why is `application/json` safer than `application/x-www-form-urlencoded`?
`application/x-www-form-urlencoded` is a "simple" content type that can be sent from a standard HTML form without JavaScript. This makes it easier to perform CSRF attacks (e.g., a hidden form on a malicious site submitting to your API). `application/json` typically requires JavaScript (XHR/Fetch) and triggers a CORS Preflight, providing stronger browser-side controls.

### 3. How do you handle `Content-Type: application/json; charset=utf-8`?
You cannot simply check `if header == "application/json"`. You must parse the header (using Go's `mime.ParseMediaType`) to separate the media type (`application/json`) from the parameters (`charset=utf-8`).

### 4. Does validating `Content-Type` prevent malicious file uploads?
No. Validating the header only checks the *label* the client put on the data. An attacker can upload a virus and label it `image/jpeg`. You must also validate the **Magic Bytes** (file signature) of the actual content stream to verify the file type.

### 5. Why should you ignore the Content-Type header for GET requests?
GET requests (conceptually) do not have a request body (payload), so the `Content-Type` header is irrelevant. Validating it on GET requests might block legitimate clients that don't send headers for simple retrievals.
