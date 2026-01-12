---
---

# Web Server Caching (Reverse Proxy)

**Web Server Caching** is implemented on the server-side infrastructure, typically using a reverse proxy or a dedicated caching engine sitting in front of the application.

## **Common Tools**
- **Nginx**: Can cache fastcgi, proxy, and static responses.
- **Varnish Cache**: An HTTP accelerator designed for content-heavy dynamic websites.

## **Pros**
- **Unified Cache**: Caches responses for all users at a single point.
- **Fast Response**: Skips the application logic entirely for cached paths.
- **Load Balancing Integration**: Often combined with load balancing features.
