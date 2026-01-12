---
tags: ['aws', 'roadmap', 'cloudfront', 'cloud']
---

# CloudFront Policies (Cache, Origin Request, Response Headers)

## Summary
CloudFront Policies are reusable configurations that define how a distribution handles caching, origin communication, and response header modification. They replace the legacy "forwarding" settings, providing a decoupled architecture that allows for high cache hit ratios while ensuring the origin receives necessary request metadata and viewers receive secure, customized responses.

## Detailed Explanation

### 1. Cache Policies
Cache policies determine what information from the viewer request is included in the **Cache Key**. The cache key is the unique identifier for an object in CloudFront's cache.
- **Cache Key Components**:
    - **Headers**: Whitelist specific headers (e.g., `Accept-Language`) to serve different versions of content based on their values.
    - **Cookies**: Whitelist specific cookies (e.g., `session-id`) if the content varies by user.
    - **Query Strings**: Whitelist parameters (e.g., `v=1.2`) to ensure unique versions are cached.
- **TTL (Time to Live) Settings**:
    - **Minimum TTL**: The lower bound for how long CloudFront caches an object (default 0).
    - **Maximum TTL**: The upper bound for caching (default 31,536,000 seconds / 1 year).
    - **Default TTL**: Applied when the origin doesn't send cache-control headers.
- **Compression**: You can enable automatic compression (Gzip/Brotli) within the cache policy, allowing CloudFront to compress objects on the fly before serving them to viewers that support it.

### 2. Origin Request Policies
These policies control which information from the viewer request is forwarded to the origin when CloudFront sends a request after a cache miss.
- **Separation of Concerns**: Unlike the Cache Policy, the Origin Request Policy does **not** affect the cache key. This allows you to forward many headers/cookies for analytics or logic without fragmenting the cache.
- **Custom Header Addition**: CloudFront can automatically inject additional headers into the origin request, such as `CloudFront-Viewer-Country`, `CloudFront-Is-Mobile-Viewer`, and `CloudFront-Viewer-ASN`.

### 3. Response Headers Policies
Response headers policies allow you to modify the HTTP headers sent from CloudFront to the viewer, regardless of what the origin returned.
- **CORS (Cross-Origin Resource Sharing)**: Centrally manage CORS headers (`Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, etc.) for multiple distributions.
- **Security Headers**: Inject vital security headers like:
    - `Strict-Transport-Security` (HSTS)
    - `Content-Security-Policy` (CSP)
    - `X-Frame-Options` (to prevent clickjacking)
    - `X-Content-Type-Options: nosniff`
- **Header Removal**: Remove sensitive or redundant headers from the origin response (e.g., `X-Powered-By`, `Server`).
- **Server-Timing**: Enables the `Server-Timing` header, providing detailed metrics in the response (e.g., `cdn-cache-hit`, `cdn-upstream-fbl`) for debugging and performance monitoring.

## Interview Questions
1. **Q: Why should you keep the 'User-Agent' header out of the Cache Policy but include it in the Origin Request Policy?**
   A: Including `User-Agent` in the Cache Policy creates a unique cache entry for every single browser string, which drastically lowers the Cache Hit Ratio. By using the Origin Request Policy, the origin gets the browser info for logic/analytics, while CloudFront maintains a single, highly-shared cache entry for the content.
2. **Q: How do Cache Policy TTL settings interact with origin 'Cache-Control' headers?**
   A: The Cache Policy's Min and Max TTL act as boundaries. If the origin's `max-age` is below the Min TTL, CloudFront uses the Min TTL. If it's above the Max TTL, CloudFront uses the Max TTL.
3. **Q: What is the primary benefit of using Response Headers Policies for CORS?**
   A: It allows developers to manage CORS settings at the CDN layer rather than requiring code changes or complex configuration on every origin server (e.g., multiple S3 buckets or EC2 instances).
4. **Q: What is the 'Server-Timing' header and how do you enable it?**
   A: It's a metric-rich header used for debugging performance. It's enabled via a Response Headers Policy by setting a sampling rate (0-100%). It shows details like whether a request hit the edge or regional cache and the origin latency.
5. **Q: What happens if you include a header in both the Cache Policy and the Origin Request Policy?**
   A: All headers in the Cache Policy are automatically forwarded to the origin. The Origin Request Policy is only needed for headers that are **not** already in the Cache Policy.
