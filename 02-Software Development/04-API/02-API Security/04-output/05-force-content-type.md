#API
---
---

# Force Content-Type

## Summary
Forcing a specific `Content-Type` header (typically `application/json` for modern APIs) is a defense against **MIME Sniffing** and **Ambiguity** attacks. If a server responds with JSON data but labels it as `text/plain` or leaves the header empty, browsers might try to "guess" the format. This can lead to security issues if the browser incorrectly identifies the content as HTML and executes malicious scripts embedded within the JSON strings (a form of XSS).

## Detailed Explanation

### 1. The Vulnerability: Ambiguous Types
Browsers are built to be tolerant of misconfigured servers. If you send a JSON response without a `Content-Type`, a browser might look at the data:
`{"user": "<script>alert(1)</script>"}`
And decide "This looks like HTML because of the tags," then render it, executing the script.

### 2. Middleware Enforcement
In Go, you should enforce the `Content-Type` on every response written by your API. This is best done via middleware or a custom `ResponseWriter` wrapper.

#### Go Implementation

```go
package main

import "net/http"

// ForceJSONMiddleware ensures every response has the correct Content-Type
func ForceJSONMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Set the header BEFORE the handler writes the body
		w.Header().Set("Content-Type", "application/json")
		
		// Optional: Add charset
		// w.Header().Set("Content-Type", "application/json; charset=utf-8")

		next.ServeHTTP(w, r)
	})
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/data", func(w http.ResponseWriter, r *http.Request) {
		// Even if the developer forgets to set the header here, the middleware handles it.
		w.Write([]byte(`{"status": "safe"}`))
	})

	http.ListenAndServe(":8080", ForceJSONMiddleware(mux))
}
```

### 3. Why `text/plain` is dangerous for JSON
Some developers use `text/plain` for JSON to avoid CORS preflight requests (since `text/plain` is a "simple" content type). However, this disables the browser's JSON parsing protections and security features that apply specifically to `application/json`. It also signals to the browser that the content is meant to be read by humans, not machines, which can lead to unexpected rendering behaviors.

## Interview Questions

### 1. What is the difference between `Content-Type` and `Accept` headers?
*   **`Content-Type`**: Tells the receiver what the data *is* (e.g., "I am sending you JSON").
*   **`Accept`**: Tells the sender what data format the client *wants* (e.g., "Please send me JSON").

### 2. If I set `X-Content-Type-Options: nosniff`, do I still need to force `Content-Type`?
Yes. `nosniff` tells the browser "Trust the Content-Type header." If you don't set the `Content-Type` header (or set it incorrectly), the browser effectively has nothing to trust, or trusts the wrong type. They work together: one sets the type, the other enforces strict adherence to it.

### 3. Can setting `Content-Type: application/json` prevent XSS?
It helps significantly. Browsers generally will not execute `<script>` tags found inside a file served with `application/json`, because that MIME type is not executable. However, it is not a silver bullet; you still need to escape user input to prevent issues in clients that might manually parse and render the HTML (like a frontend framework dangerously using `innerHTML`).

### 4. Why should you always include `charset=utf-8`?
To prevent **Charset Confusion** attacks. If an attacker can trick the browser into interpreting the response as a different encoding (like UTF-7), they might be able to bypass XSS filters that only look for `<` and `>` in ASCII/UTF-8.
