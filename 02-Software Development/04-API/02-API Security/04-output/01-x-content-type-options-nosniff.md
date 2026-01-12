#API
---
---

# X-Content-Type-Options: nosniff

## Summary
The **`X-Content-Type-Options: nosniff`** header is a critical security control that instructs web browsers to strictly adhere to the MIME types declared in the `Content-Type` header. It disables **MIME sniffing**, a browser behavior where the browser attempts to "guess" the file type by inspecting the content byte stream, ignoring the server's declared type. This prevents attackers from disguising malicious scripts as harmless file types (like images or text files) to execute Cross-Site Scripting (XSS) attacks.

## Detailed Explanation

### 1. Browser Behavior: Sniffing vs. Strict
*   **Without Header (Sniffing)**: If a server sends a file with `Content-Type: text/plain` but the file contains `<script>alert(1)</script>`, older browsers (and some modern ones in compatibility modes) might "sniff" the HTML tags and execute the script. This transforms a harmless text file upload into an XSS vector.
*   **With Header (`nosniff`)**: The browser trusts the server. If the server says `text/plain`, it is rendered as text, even if it looks like executable code.

### 2. The Attack Vector: Polyglots
An attacker can create a **Polyglot** file—a file that is valid as multiple types (e.g., valid JPEG and valid JavaScript).
1.  Attacker uploads `avatar.jpg` (which contains hidden JS).
2.  Server serves it as `Content-Type: image/jpeg`.
3.  Victim visits a page referencing this file in a script tag: `<script src="/avatar.jpg"></script>`.
4.  **Without nosniff**: The browser might ignore the image type, sniff the JS content, and execute it.
5.  **With nosniff**: The browser sees `image/jpeg` context in a script tag, realizes it mismatches `application/javascript`, and blocks execution.

### 3. Go Middleware Implementation
In Go, security headers are best applied via global middleware.

```go
package main

import (
	"fmt"
	"net/http"
)

func SecurityHeadersMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Prevent MIME sniffing
		w.Header().Set("X-Content-Type-Options", "nosniff")
		
		next.ServeHTTP(w, r)
	})
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Secure Content"))
	})

	http.ListenAndServe(":8080", SecurityHeadersMiddleware(mux))
}
```

---

## Interview Questions

### 1. Why is MIME sniffing enabled by default in browsers?
It was originally a usability feature. In the early web, servers were often misconfigured (sending everything as `text/plain` or `application/octet-stream`). Sniffing allowed browsers to correctly render images and HTML despite server errors. Security was a secondary concern at the time.

### 2. Does `X-Content-Type-Options: nosniff` prevent all XSS?
No. It only prevents XSS caused by **MIME confusion** (e.g., executing an image as a script). It does not stop XSS vulnerabilities in your actual application code (like unescaped user input in HTML).

### 3. What happens if you send `nosniff` but forget the `Content-Type` header?
The browser may block the resource entirely or treat it as `text/plain`. Since it is forbidden from guessing, and it hasn't been told what the content is, the "safest" default is often to simply download it or display raw text, potentially breaking the site's styling or functionality.

### 4. How does `nosniff` relate to `Content-Type` validation?
They are complementary.
*   **Validation** (Input): Ensures the server only *accepts* correct types (e.g., rejecting an upload named `file.js` sent as `image/png`).
*   **`nosniff`** (Output): Ensures the browser *treats* the response strictly as the server declared it.
