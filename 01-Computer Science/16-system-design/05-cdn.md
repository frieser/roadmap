---
---

## Summary
A **CDN (Content Delivery Network)** is a geographically distributed group of servers that cache content close to end users. It allows a user in Tokyo to download images from a server in Tokyo, rather than fetching them from your origin server in New York.

## Detailed Explanation
### How it Works
1.  **Edge Servers**: Servers located at the "edge" of the network (IXPs, ISPs).
2.  **Pull Zone**: When a user requests `image.png`, the Edge checks its cache. If missing, it "pulls" it from your Origin, caches it, and serves it.
3.  **Push Zone**: You manually upload content to the CDN storage.

### Benefits
*   **Latency**: Reduced RTT (Round Trip Time).
*   **Bandwidth**: Offloads traffic from your origin server (cheaper).
*   **Availability**: DDoS protection and redundancy.

### Go Context
Go apps typically serve static assets via a CDN. The HTML templates render URLs pointing to the CDN (`https://cdn.example.com/style.css`) instead of the local server.

## Interview Questions
**Q: How do you invalidate a file on CDN (e.g., you updated style.css)?**
A: 
1.  **Purge**: Send a command to CDN to delete the file (slow, can take minutes).
2.  **Versioning**: Change the URL (`style.v2.css` or `style.css?v=2`). This forces the CDN to treat it as a new object (instant).

**Q: Can CDNs cache dynamic content (API responses)?**
A: Yes, if configured with proper Cache-Control headers, but it's risky. Typically CDNs are used for static assets (Images, CSS, JS, Video).

## Diagram
```mermaid
graph TD
    User[User in London]
    Edge[CDN Edge London]
    Origin[Origin Server NY]
    
    User -->|Request| Edge
    Edge --|Cache Miss| Origin
    Origin --|Return File| Edge
    Edge --|Serve & Cache| User
    
    User2[User 2 in London] -->|Request| Edge
    Edge --|Cache Hit| User2
```
