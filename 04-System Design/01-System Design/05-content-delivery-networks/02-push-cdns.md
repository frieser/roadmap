---
---

## Summary
A **Push CDN** is an active strategy where you are responsible for uploading (pushing) content to the CDN's storage servers before users can access it. The content resides on the CDN permanently until deleted, acting as a primary storage rather than just a cache.

## Detailed Explanation

### Workflow
1.  **Deployment**: Your CI/CD pipeline or backend application uploads `video.mp4` to the CDN storage (via FTP, API, or S3-like interface).
2.  **Propagation**: The CDN distributes the file to its edge nodes (or central storage accessible by edges).
3.  **User Request**: Users request the file.
4.  **Serve**: The CDN serves the file immediately. There is no concept of a "Cache Miss" fetching from your server; if it's not on the CDN, it's a 404.

### Pros and Cons
| Pros | Cons |
| :--- | :--- |
| **Performance**: Zero latency for the first request; content is pre-warmed. | **Maintenance**: Requires explicit upload/delete workflows. |
| **Reliability**: Content is available even if your main server goes offline. | **Storage Cost**: You pay to store everything, even rarely accessed files. |
| **Control**: You decide exactly what is cached and when it expires. | **Complexity**: Integration into deployment pipelines is mandatory. |

### Use Cases
*   **Software Downloads**: Distributing a 5GB game patch or installer.
*   **Video on Demand (VoD)**: Streaming libraries (Netflix model) where assets are static and massive.
*   **Critical Assets**: Core branding files that must never fail to load.

## Go Context
In a Push CDN scenario, your Go application needs an "Uploader" service.

```go
// The application proactively pushes content to the CDN
func UploadToCDN(file []byte, filename string) error {
    // Conceptual example: pushing to an S3 bucket configured as a Push CDN source
    _, err := s3Client.PutObject(&s3.PutObjectInput{
        Bucket: aws.String("my-push-cdn-bucket"),
        Key:    aws.String(filename),
        Body:   bytes.NewReader(file),
        ACL:    aws.String("public-read"),
    })
    return err
}
```

## Interview Questions

### Q: Why would you use a Push CDN for a large file download?
**A:** Large files take a long time to "pull" from an origin on the first request, leading to a timeout or bad experience for the first user. Pushing ensures the file is fully ready and replicated at the edge before the first user clicks "Download."

### Q: Can you mix Push and Pull?
**A:** Yes. A common pattern is to use **Push** for critical, heavy assets (like the Hero Video on a homepage) to guarantee performance, while using **Pull** for the thousands of small, long-tail images in a catalog to save on storage and workflow complexity.
