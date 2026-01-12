#API
---
---

## Summary
HTTP Caching is a mechanism where the browser or intermediate proxies store copies of responses to fulfill subsequent requests. This reduces server load, saves bandwidth, and significantly improves performance by avoiding redundant network round-trips. Caching is controlled primarily through HTTP response headers that define expiration and validation strategies.

## Detailed Explanation

### 1. Cache-Control Header
The \`Cache-Control\` header is the primary tool for defining caching policies. It can contain multiple directives:

*   **\`max-age=<seconds>\`**: Defines the TTL (Time To Live). The resource is considered "fresh" for this duration.
*   **\`no-cache\`**: The response can be cached, but it **must be validated** with the origin server before reuse (using ETags or Last-Modified).
*   **\`no-store\`**: The response **must not be cached** at all. Used for sensitive data.
*   **\`public\`**: The response can be cached by any cache (Browser, CDN, Proxy).
*   **\`private\`**: The response is intended for a single user and should only be stored in a private (browser) cache.
*   **\`must-revalidate\`**: Once a resource becomes stale, it must not be served without successful validation.

### 2. Validators: ETag and Last-Modified
When a cached response becomes "stale" (exceeds \`max-age\`), the client uses validators to check if the content has changed.

*   **ETag (Entity Tag)**:
    *   A unique string (often a hash) representing the specific version of a resource.
    *   **Flow**: Server sends \`ETag: "v1"\`. Client stores it. Next time, client sends \`If-None-Match: "v1"\`. If unchanged, server returns \`304 Not Modified\`.
*   **Last-Modified**:
    *   A timestamp of when the resource was last updated.
    *   **Flow**: Server sends \`Last-Modified: <date>\`. Client sends \`If-Modified-Since: <date>\`. If unchanged, server returns \`304 Not Modified\`.

### 3. Caching Tiers
*   **Client-side (Browser)**: Individual user cache. Fastest access.
*   **Proxy Caching**: Shared caches in the network path (ISPs, corporate firewalls).
*   **Server-side (Reverse Proxy/CDN)**: Managed caches like Cloudflare, Akamai, or Nginx/Varnish that sit in front of your origin server to offload traffic.

### 4. Cache Busting
Since long-lived caches (\`max-age=31536000\`) can prevent users from seeing updates, developers use **Cache Busting** by changing the URL when content changes (e.g., \`style.css?v=2\` or \`app.d41d8cd9.js\`).

---

## Go Implementation: ETag and Conditional Requests

In Go, you can manually implement ETag validation to save bandwidth and processing power.

\`\`\`go
package main

import (
	"crypto/md5"
	"fmt"
	"io"
	"net/http"
)

func cachedHandler(w http.ResponseWriter, r *http.Request) {
	content := "This is some dynamic but rarely changing content."
	
	// 1. Generate ETag (e.g., MD5 hash of content)
	h := md5.New()
	io.WriteString(h, content)
	etag := fmt.Sprintf("\"%x\"", h.Sum(nil))

	// 2. Set Cache-Control and ETag headers
	w.Header().Set("Cache-Control", "public, max-age=3600")
	w.Header().Set("ETag", etag)

	// 3. Handle Conditional Request (Check If-None-Match)
	if r.Header.Get("If-None-Match") == etag {
		w.WriteHeader(http.StatusNotModified)
		return
	}

	// 4. Send full response if ETag doesn't match
	w.WriteHeader(http.StatusOK)
	w.Write([]byte(content))
}

func main() {
	http.HandleFunc("/api/data", cachedHandler)
	http.ListenAndServe(":8080", nil)
}
\`\`\`

---

## Interview Questions

**Q: What is the difference between \`no-cache\` and \`no-store\`?**
**A:** \`no-cache\` allows the response to be stored but requires the client to revalidate with the server before every use. \`no-store\` forbids the response from being stored in any cache at all.

**Q: What does a \`304 Not Modified\` status code mean?**
**A:** It means the client's cached version is still valid. The server sends this empty response (no body) to save bandwidth when the \`If-None-Match\` or \`If-Modified-Since\` headers match the current server state.

**Q: How do you handle caching for a Single Page Application (SPA)?**
**A:** Usually, the \`index.html\` is served with \`Cache-Control: no-cache\` (to ensure users always get the latest entry point), while static assets (JS, CSS, Images) are served with a long \`max-age\` and unique hashes in their filenames for cache busting.

**Q: When should you use \`Cache-Control: private\`?**
**A:** When the response contains user-specific data (like a profile page) that should not be stored in shared caches like CDNs or proxies, but is safe to store in the user's local browser.
