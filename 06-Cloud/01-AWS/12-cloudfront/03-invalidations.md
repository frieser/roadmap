#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'cloudfront']
---

## Summary
CloudFront Invalidations allow you to remove objects from the edge caches before their TTL (Time To Live) expires. This is essential when you need to update content immediately across the global network without waiting for the cache to naturally refresh.

## Detailed Explanation

### Mechanism
When you submit an invalidation request, CloudFront removes the specified files from its edge locations. The next time a user requests the file, CloudFront goes back to the origin to fetch the latest version. This is a "pull" model where the cache is cleared, and the next request triggers a refresh.

### Invalidating Paths
You can specify which objects to invalidate using paths:
- **Specific File**: `/images/logo.png`
- **Directory**: `/images/*` (invalidates all files in that path)
- **Everything**: `/*` (invalidates the entire distribution)

### Costs
- **Free Tier**: The first **1,000 invalidation paths** per month are free.
- **Paid Tier**: Above 1,000, each path costs **$0.005**.
- **Efficiency**: A path with a wildcard (e.g., `/*`) counts as a **single path**, even if it invalidates millions of files. This makes wildcards very cost-effective for large-scale updates.

### Object Versioning (The Alternative)
Instead of invalidating, AWS recommends **Object Versioning**. This involves changing the filename or path for every update (e.g., `style.css` -> `style.v2.css` or `v2/style.css`).
- **Pros**: 
    - No cost for invalidations.
    - Immediate update (new requests hit the new URL instantly).
    - Easy rollbacks (just point back to the previous version).
- **Cons**: Requires updating links in the application code or HTML.

### Bash Example
Using the AWS CLI to manage invalidations:

```bash
# Invalidate all files in a distribution
aws cloudfront create-invalidation \
  --distribution-id E123456789ABCD \
  --paths "/*"

# Check the status of an invalidation request
# (Status will be 'InProgress' or 'Completed')
aws cloudfront get-invalidation \
  --id I1234567890ABCD \
  --distribution-id E123456789ABCD
```

## Interview Questions

1. **Q: What is the difference between Invalidation and Object Versioning?**
   **A:** Invalidation removes the old object from cache, keeping the same URL. Versioning changes the URL to point to a new object. Versioning is generally preferred because it is free and provides immediate updates without cache propagation delay.

2. **Q: How does CloudFront charge for invalidations?**
   **A:** CloudFront offers 1,000 free invalidation paths per month. Beyond that, it costs $0.005 per path. Wildcard paths count as a single path regardless of the number of files affected.

3. **Q: How long does an invalidation take?**
   **A:** It typically takes a few minutes (usually 1-5 minutes) to propagate the invalidation request to all edge locations worldwide.

4. **Q: Can you cancel an invalidation request?**
   **A:** No, once an invalidation request is submitted, it cannot be cancelled or modified. You must wait for it to complete.

5. **Q: Does invalidating a path also clear the cache in regional edge caches?**
   **A:** Yes, invalidation requests are sent to all edge locations and regional edge caches to ensure the content is cleared globally.
