---
---

## Summary
A Content Delivery Network (CDN) is a geographically distributed group of servers that work together to provide fast delivery of Internet content. A CDN allows for the quick transfer of assets needed for loading Internet content including HTML pages, javascript files, stylesheets, images, and videos.

## Detailed Explanation
CDNs work by caching content at the "edge" of the network, closer to the users.

### How it works
1. A user requests a resource (e.g., an image).
2. The DNS routes the request to the nearest CDN "Point of Presence" (PoP).
3. If the PoP has the file (a "Cache Hit"), it serves it immediately.
4. If not (a "Cache Miss"), it fetches the file from the "Origin" server, caches it, and then serves it to the user.

### Key Benefits
- **Reduced Latency**: Physical proximity to users means faster load times.
- **Reduced Origin Load**: Most traffic is handled by the CDN, saving server resources.
- **Improved Reliability**: Distributed nature protects against DDoS attacks and hardware failures.
- **Bandwidth Savings**: CDNs often have lower bandwidth costs than cloud origin servers.

### CDN Concepts
- **Edge Servers**: The distributed servers that serve content.
- **Origin Server**: Your actual application server where the "source of truth" resides.
- **TTL (Time to Live)**: How long the CDN should keep a file before checking with the origin for updates.
- **Purging**: Manually telling the CDN to delete its cached version of a file.

## Go Context
As a backend developer, you interact with CDNs via HTTP headers.

### Example: Setting Cache-Control headers in Go
```go
func imageHandler(w http.ResponseWriter, r *http.Request) {
	// Tell the CDN to cache this for 1 hour
	w.Header().Set("Cache-Control", "public, max-age=3600")
	// serve the image...
}
```

## Interview Questions
- **Q: What is the difference between a Pull CDN and a Push CDN?**
- **A:** In a Pull CDN, the edge server "pulls" the content from the origin when it's first requested. In a Push CDN, you manually upload content to the CDN's storage before it's requested.

- **Q: How do you handle versioning with CDN cached assets?**
- **A:** The best practice is "Cache Busting" — including a hash or version number in the filename (e.g., `style.v2.css`). Since the filename changes, the CDN treats it as a new resource and ignores the old cached version.

- **Q: What is "Stale-While-Revalidate"?**
- **A:** It's a `Cache-Control` directive that tells the CDN to serve a stale (expired) version of a resource while it fetches an updated version in the background. This improves performance by avoiding "Cache Miss" delays for the user.
