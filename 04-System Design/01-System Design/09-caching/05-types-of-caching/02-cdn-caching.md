---
---

# CDN (Content Delivery Network) Caching

**CDN Caching** (Edge Caching) involves distributing copies of content to geographically dispersed servers (Edge Points of Presence - PoPs).

## **How it Works**
- When a user requests a resource, the request is routed to the nearest CDN server.
- If the CDN server has the content (Cache Hit), it serves it directly.
- If not (Cache Miss), the CDN fetches it from the **Origin Server**, caches it, and then serves it.

## **Pros**
- **Reduced Latency**: Content is physically closer to the user.
- **Scalability**: Offloads massive amounts of traffic from the origin server (Origin Shielding).
- **DDoS Protection**: CDNs can absorb large-scale traffic spikes.

## **Go Context: Integration**
Usually managed via infrastructure configuration (Cloudflare, CloudFront), but Go applications must send correct HTTP headers to allow CDNs to cache responses.

```go
func handleStaticAsset(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Cache-Control", "public, max-age=31536000") // 1 year
    // serve file...
}
```
