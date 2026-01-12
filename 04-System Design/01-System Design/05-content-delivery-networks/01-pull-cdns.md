---
---

## Summary
A **Pull CDN** (or Origin Pull) is a passive strategy where the CDN automatically retrieves content from your origin server only when a user requests it. It relies on the "Cache Miss" mechanism to populate the edge servers. This is the most common configuration for general websites.

## Detailed Explanation

### Workflow
1.  **User Request**: A user in London requests `logo.png` from `cdn.yoursite.com`.
2.  **CDN Check**: The London edge server checks its local cache.
3.  **Cache Miss**: The file is not there (or expired).
4.  **Origin Fetch**: The CDN requests `logo.png` from your Origin Server (e.g., in New York).
5.  **Serve & Cache**: The CDN serves the file to the user and saves a copy in London for future requests.

### Pros and Cons
| Pros | Cons |
| :--- | :--- |
| **Simple Setup**: No code changes required; just point DNS. | **First Request Latency**: The first user (or after expiry) waits longer (TTFB). |
| **Low Maintenance**: Automatically mirrors your origin structure. | **Origin Traffic**: Redundant requests if content expires frequently. |
| **Efficient Storage**: Only popular content is stored on edge nodes. | **Origin Dependency**: If Origin is down, cache misses result in errors. |

### Use Cases
*   **Websites**: Serving images, CSS, and JS for blogs or e-commerce sites.
*   **User Generated Content**: Profile pictures where you can't predict what will be popular.
*   **Variable Traffic**: Sites where content popularity spikes and fades quickly.

## Go Context
In a Go application using a Pull CDN, you typically don't need special logic. You just ensure your HTML templates point to the CDN URL.

```go
// The application doesn't upload anything.
// It just constructs URLs pointing to the CDN.
func GetImageURL(imageName string) string {
    // "https://cdn.example.com" is a Pull CDN pointing to our server
    return fmt.Sprintf("https://cdn.example.com/images/%s", imageName)
}
```

## Interview Questions

### Q: What is "Cache Busting" in a Pull CDN?
**A:** Since Pull CDNs rely on TTL (Time To Live), updating a file on the origin doesn't immediately update the CDN. **Cache Busting** involves changing the filename (e.g., `style.v2.css` or `style.css?v=2`) to force the CDN to treat it as a new "Miss" and fetch the new version immediately.

### Q: How does TTL affect a Pull CDN?
**A:** A short TTL ensures freshness but increases load on the origin (more fetches). A long TTL reduces origin load but risks serving stale content. The balance depends on how often the content changes.
