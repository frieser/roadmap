#API
---
---

# X-Frame-Options: DENY

## Summary
The **`X-Frame-Options`** HTTP response header is a defensive mechanism used to prevent **Clickjacking** attacks. Clickjacking occurs when an attacker overlays a transparent iframe of your website on top of a malicious page. Users think they are clicking a button on the attacker's site (like "Claim Prize"), but they are actually clicking a sensitive button on your site (like "Delete Account" or "Transfer Funds") loaded invisibly in the background.

*   **Header**: `X-Frame-Options: DENY` (or `SAMEORIGIN`)
*   **Modern Replacement**: `Content-Security-Policy: frame-ancestors` (but XFO is still required for depth/legacy support).

## Detailed Explanation

### 1. Directives: DENY vs. SAMEORIGIN
*   **`DENY`**: The page cannot be displayed in a frame, regardless of the site attempting to do so. This is the most secure setting and recommended for all pages unless framing is explicitly required.
*   **`SAMEORIGIN`**: The page allows itself to be framed only by pages on the same origin (same domain, protocol, and port). Useful if your application uses internal iframes.

### 2. Why APIs Need It
While purely data-driven JSON APIs (`application/json`) are rarely rendered in iframes, many APIs serve:
*   HTML Error pages (404/500).
*   OAuth2 Consent screens.
*   Admin Panels / Dashboards.
*   Swagger/OpenAPI documentation UIs.

If these HTML interfaces are vulnerable to clickjacking, an attacker could force an admin to revoke keys or delete data.

### 3. Go Middleware Implementation
Applying this header globally ensures that even accidental HTML responses are protected.

```go
package main

import (
	"net/http"
)

func ClickjackingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Strict protection: No framing allowed
		w.Header().Set("X-Frame-Options", "DENY")
		
		// Modern CSP equivalent (optional but recommended)
		// w.Header().Set("Content-Security-Policy", "frame-ancestors 'none';")

		next.ServeHTTP(w, r)
	})
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/login", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Login Form"))
	})

	http.ListenAndServe(":8080", ClickjackingMiddleware(mux))
}
```

---

## Interview Questions

### 1. Why is `X-Frame-Options` needed if we have `Content-Security-Policy`?
CSP's `frame-ancestors` directive is the modern standard and is more flexible (allowing multiple specific domains). However, older browsers (like Internet Explorer 11) do not support CSP `frame-ancestors` but do support `X-Frame-Options`. Using both provides **Defense in Depth**.

### 2. Does `X-Frame-Options` protect against XSS?
No. It specifically protects against **UI Redressing** (Clickjacking). XSS involves injecting scripts; Clickjacking involves manipulating the visual layering of the page.

### 3. If I set `X-Frame-Options: DENY`, can I still use `<iframe>` on my own site to load my own content?
No. `DENY` blocks *all* framing, even from the same domain. If you need to frame your own content (e.g., a modal dialog loaded via iframe), you must use `SAMEORIGIN`.

### 4. How does `frame-ancestors` improve upon `X-Frame-Options`?
`X-Frame-Options` allows only `DENY` or `SAMEORIGIN`. You cannot whitelist a specific partner domain (e.g., `trustedsite.com`). CSP `frame-ancestors` allows granular whitelisting: `frame-ancestors 'self' https://trustedsite.com`.
