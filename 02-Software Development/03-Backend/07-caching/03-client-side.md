---
---

## Summary
Client-side caching refers to the browser storing copies of web resources (like HTML, CSS, JS, and images) locally on the user's device. This prevents the browser from having to download the same files multiple times, significantly improving page load speeds and reducing server bandwidth.

## Detailed Explanation
Client-side caching is primarily controlled by the server via HTTP response headers.

### Key HTTP Headers
- **`Cache-Control`**: The modern standard.
    - `max-age`: How long (in seconds) the resource is considered fresh.
    - `no-store`: Do not cache anything.
    - `no-cache`: Cache the resource but must revalidate with the server before using it.
    - `public`/`private`: Whether intermediate caches (like CDNs) can store the file.
- **`ETag`**: A unique identifier for a specific version of a resource (usually a hash). The browser sends this back in an `If-None-Match` header to see if the file has changed.
- **`Last-Modified`**: The date the file was last changed. Used with `If-Modified-Since`.

### Revalidation Process
1. Browser wants `logo.png`. It has a cached version but it's "stale" (expired).
2. Browser sends a request to the server with `If-None-Match: "hash123"`.
3. If the server's version is the same, it returns `304 Not Modified` (with no body).
4. Browser uses its local copy. This saves bandwidth even if it doesn't save a round trip.

## Go Context
Setting these headers correctly is a key responsibility of the backend.

### Example: Implementing ETags in Go
```go
func handler(w http.ResponseWriter, r *http.Request) {
    content := "Hello, World!"
    etag := `"v1-hash"`

    if r.Header.Get("If-None-Match") == etag {
        w.WriteHeader(http.StatusNotModified)
        return
    }

    w.Header().Set("ETag", etag)
    w.Header().Set("Cache-Control", "public, max-age=3600")
    fmt.Fprint(w, content)
}
```

## Interview Questions
- **Q: What is the difference between `no-cache` and `no-store`?**
- **A:** `no-store` tells the browser never to save the resource at all (best for sensitive data). `no-cache` tells the browser it *can* save the resource, but it *must* check with the server (revalidate) before using it.

- **Q: What happens if a server returns a `304 Not Modified` response?**
- **A:** The response body is empty. The browser sees this status code and knows that its locally cached version is still valid, so it renders the resource from its own disk/memory.

- **Q: Why use ETags instead of Last-Modified?**
- **A:** ETags are more precise. `Last-Modified` only has 1-second resolution, which might be an issue for files changing very rapidly. Also, a file's timestamp might change even if its content hasn't; an ETag (hash) would remain the same in that case.
