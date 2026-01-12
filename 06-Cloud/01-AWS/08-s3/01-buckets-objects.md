#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 's3']
---

## Summary
Amazon S3 (Simple Storage Service) is an object storage service that offers industry-leading scalability, data availability, security, and performance. At its core, S3 is a **Key-Value store** where data is stored as **Objects** within **Buckets**. It provides a flat hierarchy where the "Key" (filename/path) identifies the "Value" (data). S3 operates on a regional basis but requires a globally unique name for each bucket.

## Detailed Explanation

### 1. The Key-Value Store Model
Unlike a traditional file system with a nested directory structure, S3 is a flat storage system.
- **Buckets**: Containers for objects. Every object is contained in a bucket.
- **Objects**: The fundamental entities stored in S3. An object consists of:
    - **Key**: The "name" assigned to an object (e.g., `images/logo.png`). It can be up to 1024 bytes long.
    - **Value**: The actual data (from 0 bytes to 5 TB).
    - **Version ID**: Used for version control.
    - **Metadata**: A set of name-value pairs.
    - **Tags**: Key-value pairs for categorization.

### 2. Bucket Naming Rules
Buckets have a **global namespace**, meaning a bucket name must be unique across all AWS accounts globally.
- **Length**: 3 to 63 characters.
- **Characters**: Lowercase letters, numbers, dots (`.`), and hyphens (`-`).
- **Formatting**: Must start and end with a letter or number.
- **Restrictions**: No underscores, no uppercase letters, and cannot be formatted as an IP address.

### 3. Strong Consistency Model
As of December 2020, Amazon S3 provides **strong read-after-write consistency** for all applications.
- **New Objects (PUT)**: After a successful `PUT` of a new object, any subsequent `GET` or `LIST` request will immediately see the object.
- **Overwrites and Deletes**: If you overwrite an existing object or delete one, subsequent requests will immediately show the latest version or the deletion.
- **Performance**: This consistency is achieved without a performance penalty or additional cost.

### 4. Metadata and Tags
- **System Metadata**: Automatically maintained by S3 (e.g., `Last-Modified`, `ETag`, `Content-Type`).
- **User-Defined Metadata**: Custom metadata provided during upload, prefixed with `x-amz-meta-`. These are stored with the object and cannot be modified after upload (requires a re-upload or copy).
- **Object Tags**: Up to 10 tags per object. Unlike metadata, tags can be modified independently of the object data and are often used for IAM policies, Lifecycle rules, and cost allocation.

### 5. Go Example: PutObject
Using the AWS SDK for Go v2, you can upload an object to an S3 bucket as follows:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"strings"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/s3"
)

func main() {
	// Load AWS SDK configuration
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	// Create an S3 client
	client := s3.NewFromConfig(cfg)

	bucketName := "my-unique-bucket-name"
	objectKey := "docs/hello-world.txt"
	content := "Hello from Antigravity!"

	// PutObject request
	_, err = client.PutObject(context.TODO(), &s3.PutObjectInput{
		Bucket:      aws.String(bucketName),
		Key:         aws.String(objectKey),
		Body:        strings.NewReader(content),
		ContentType: aws.String("text/plain"),
		Metadata: map[string]string{
			"Author": "Antigravity",
		},
	})

	if err != nil {
		log.Fatalf("failed to upload object, %v", err)
	}

	fmt.Printf("Successfully uploaded %s to %s\n", objectKey, bucketName)
}
```

## Interview Questions

**Q: What is the maximum size of a single object in S3, and how should large files be uploaded?**
**A:** The maximum size of a single object is 5 TB. However, a single `PUT` operation can only upload up to 5 GB. For objects larger than 5 GB (and recommended for objects larger than 100 MB), you should use **Multipart Upload**, which breaks the file into smaller parts uploaded in parallel.

**Q: Can you change the region of an existing S3 bucket?**
**A:** No. The region is assigned at bucket creation and cannot be changed. To move a bucket's content to a different region, you must create a new bucket in the target region and migrate the data (e.g., using S3 Batch Operations or Replication).

**Q: How does S3's consistency model affect concurrent updates to the same object?**
**A:** S3 provides strong consistency. If two `PUT` requests are made to the same key simultaneously, S3 uses **Last-Writer-Wins** semantics. The request with the latest timestamp (determined by S3) will be the one that persists.

**Q: What is the difference between Metadata and Tags in S3?**
**A:** Metadata is uploaded with the object and becomes part of the object's header; user-defined metadata cannot be changed without replacing the object. Tags are separate from the object data, can be updated at any time without modifying the object, and are limited to 10 per object.
