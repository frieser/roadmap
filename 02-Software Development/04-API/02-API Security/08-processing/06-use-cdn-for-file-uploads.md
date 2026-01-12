#API #Security #Performance
---
---

# Use CDN for File Uploads

## Summary
Directly uploading files to an API server is an anti-pattern that consumes thread resources, bandwidth, and exposes the server to DoS attacks. The secure and performant alternative is to use **Presigned URLs** (AWS S3, Azure Blob) to allow clients to upload directly to object storage. This offloads traffic to a **CDN/Storage** provider that can handle massive scale and perform malware scanning at the edge.

## Detailed Explanation

### 1. The Problem with Direct Uploads
*   **Blocking Threads**: Uploading a 1GB file on a slow connection occupies a server thread/connection for minutes. This can starve the thread pool (Thread Exhaustion), preventing other users from accessing the API.
*   **Bandwidth Saturation**: It saturates the API's network interface.
*   **Attack Surface**: Large multipart parsers are complex and historically prone to buffer overflow or memory exhaustion attacks.

### 2. The Solution: Presigned URLs
The API generates a temporary, cryptographically signed URL that grants permission to upload *one specific file* to *one specific location* for a *short time* (e.g., 5 minutes).
1.  **Client** requests upload permission (`POST /api/files/upload-request`).
2.  **API** validates permissions and generates a Presigned URL.
3.  **Client** uploads the file directly to S3 using the Presigned URL.
4.  **S3** triggers an event (Lambda) to scan the file for malware.

### 3. Isolation & XSS
Serving user-uploaded content from the same domain as the application (`app.com`) allows **XSS**. If a user uploads `malicious.html` and another user visits `app.com/files/malicious.html`, the script executes in the context of `app.com` and can steal cookies.
*   **Fix**: Always serve user content from a separate domain (e.g., `usercontent-cdn.com`). The **Same-Origin Policy** prevents scripts on the CDN domain from accessing `app.com` data.

---

## Go (Golang) Application

### Generating S3 Presigned URLs
Using `github.com/aws/aws-sdk-go-v2`.

```go
package main

import (
	"context"
	"log"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/s3"
)

func GenerateUploadURL(bucket, key string) (string, error) {
	// 1. Load AWS Config
	cfg, err := config.LoadDefaultConfig(context.TODO())
	if err != nil {
		return "", err
	}

	// 2. Create S3 Client and Presigner
	client := s3.NewFromConfig(cfg)
	presignClient := s3.NewPresignClient(client)

	// 3. Generate the URL
	// Allow PUT method, valid for 15 minutes
	req, err := presignClient.PresignPutObject(context.TODO(), &s3.PutObjectInput{
		Bucket: aws.String(bucket),
		Key:    aws.String(key), // Use UUIDs for keys, not user filenames!
	}, func(opts *s3.PresignOptions) {
		opts.Expires = 15 * time.Minute
	})

	if err != nil {
		return "", err
	}

	return req.URL, nil
}

func main() {
	url, _ := GenerateUploadURL("my-app-uploads", "user-123/avatar.png")
	log.Println("Upload to:", url)
}
```

---

## Interview Questions

**Q1: Why should you avoid handling file uploads in your main API server?**
**A:** It causes **Resource Exhaustion**. Handling large streams keeps connections open for a long time (Slowloris effect), blocking threads that should be serving fast JSON requests. It also complicates scaling (stateful streams) and exposes the core infrastructure to parsing vulnerabilities.

**Q2: How do Presigned URLs improve security?**
**A:** They implement **Least Privilege**. The client gets permission to upload *only* that specific file to that specific key. No long-term AWS credentials are shared with the client. The URL also expires quickly, limiting the attack window.

**Q3: Why must user-uploaded content be served from a different domain (e.g., `googleusercontent.com`)?**
**A:** To prevent **Cross-Site Scripting (XSS)**. If a user uploads an HTML file with JS and opens it, the script runs on the origin serving the file. If that origin is your main app (`app.com`), the script can steal session cookies. By using a distinct domain, the **Same-Origin Policy** sandboxes the malicious script.

**Q4: How can you scan files for malware if they bypass your server?**
**A:** By using **Event-Driven Architecture**. Configure the storage bucket (S3) to trigger a function (AWS Lambda) on the `ObjectCreated` event. This function scans the file and, if clean, moves it to a "public" bucket. If malicious, it deletes the file and alerts the admins.
