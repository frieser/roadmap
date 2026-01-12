---
---

# Client Caching

**Client Caching** happens at the user's end, most commonly within the web browser or a mobile application. It aims to prevent redundant network requests for static assets or data that hasn't changed.

## **Mechanisms**
- **HTTP Headers**:
    - `Cache-Control`: Specifies directives like `max-age`, `no-cache`, `public`, `private`.
    - `ETag` (Entity Tag): A unique identifier for a version of a resource. Used for conditional requests (`If-None-Match`).
    - `Last-Modified`: Used for conditional requests (`If-Modified-Since`).
- **Local Storage / Session Storage**: Browser APIs used to store small amounts of data manually.
- **IndexedDB**: A low-level API for client-side storage of significant amounts of structured data.

## **Pros**
- **Zero Latency**: Assets are loaded instantly from the local disk/memory.
- **Bandwidth Savings**: Reduces data usage for both the client and the server.
- **Offline Support**: Enables applications to function partially or fully without an internet connection (using Service Workers).

## **Cons**
- **Stale Content**: If not managed correctly, users might see outdated versions of the site.
- **Limited Control**: Once an asset is cached on a client's device with a long TTL, it's difficult for the server to "force" an update (requires Cache Busting).
