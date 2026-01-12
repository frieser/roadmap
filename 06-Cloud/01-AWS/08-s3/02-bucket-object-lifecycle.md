#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 's3']
---

## Summary
**S3 Lifecycle Policies** are a set of rules that automate the management of objects stored in Amazon S3 buckets. These policies enable cost optimization by automatically **transitioning** objects to lower-cost storage classes or **expiring** (deleting) them based on predefined criteria such as age or versioning status.

## Detailed Explanation

### **Core Concepts**
S3 Lifecycle configurations consist of one or more rules, each containing:
- **Filter**: Defines the objects the rule applies to (e.g., prefix, tags, or entire bucket).
- **Status**: Enabled or Disabled.
- **Actions**: Transition or Expiration.

### **1. Transition Actions**
Transition actions define when objects should move to another storage class. This is the primary mechanism for cost optimization.
- **Common Paths**: `Standard` -> `Standard-IA` -> `One Zone-IA` -> `Glacier Instant Retrieval` -> `Glacier Flexible Retrieval` -> `Glacier Deep Archive`.
- **Intelligent-Tiering**: You can also transition objects to `S3 Intelligent-Tiering` to let AWS handle the transitions between access tiers automatically.
- **Constraints**: Some transitions have minimum storage duration requirements (e.g., 30 days in Standard before moving to Standard-IA).

### **2. Expiration Actions**
Expiration actions define when objects should be deleted.
- **Current Versions**: Deletes the current version of an object (creates a delete marker in versioned buckets).
- **Non-current Versions**: Permanently deletes older versions of an object after a specified number of days.
- **Expired Object Delete Markers**: Cleans up delete markers that no longer have associated non-current versions.
- **Incomplete Multipart Uploads**: Automatically aborts and deletes parts of failed or abandoned multipart uploads to save space.

### **3. Versioning and Lifecycle**
Lifecycle policies are particularly powerful when combined with **S3 Versioning**:
- You can keep non-current versions for a short period (e.g., 30 days) for recovery purposes and then transition or expire them.
- This prevents storage costs from spiraling out of control due to multiple versions of large files.

### **Go Implementation (AWS SDK v2)**
The following example demonstrates how to create a lifecycle configuration using the AWS SDK for Go v2.

```go
package main

import (
	"context"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/s3"
	"github.com/aws/aws-sdk-go-v2/service/s3/types"
)

func main() {
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := s3.NewFromConfig(cfg)

	bucketName := "my-data-bucket"

	input := &s3.PutBucketLifecycleConfigurationInput{
		Bucket: aws.String(bucketName),
		LifecycleConfiguration: &types.BucketLifecycleConfiguration{
			Rules: []types.LifecycleRule{
				{
					ID:     aws.String("MoveToIAAndThenArchive"),
					Status: types.ExpirationStatusEnabled,
					Filter: &types.LifecycleRuleFilterMemberPrefix{
						Value: "logs/", // Apply only to objects in logs/ prefix
					},
					Transitions: []types.Transition{
						{
							Days:          aws.Int32(30),
							StorageClass: types.TransitionStorageClassStandardIa,
						},
						{
							Days:          aws.Int32(90),
							StorageClass: types.TransitionStorageClassGlacier,
						},
					},
					Expiration: &types.LifecycleExpiration{
						Days: aws.Int32(365), // Expire after 1 year
					},
				},
				{
					ID:     aws.String("CleanupOldVersions"),
					Status: types.ExpirationStatusEnabled,
					Filter: &types.LifecycleRuleFilterMemberPrefix{
						Value: "", // Apply to all objects
					},
					NoncurrentVersionExpiration: &types.NoncurrentVersionExpiration{
						NoncurrentDays: aws.Int32(30), // Expire non-current versions after 30 days
					},
				},
			},
		},
	}

	_, err = client.PutBucketLifecycleConfiguration(context.TODO(), input)
	if err != nil {
		log.Fatalf("failed to put bucket lifecycle configuration, %v", err)
	}

	log.Println("Successfully applied lifecycle configuration")
}
```

## Interview Questions

**Q: What is the difference between a Transition action and an Expiration action in S3?**
**A:** Transition actions move objects between storage classes (e.g., from Standard to Glacier) to save costs while keeping the data. Expiration actions permanently delete objects or their versions after a specified period.

**Q: How do S3 Lifecycle policies handle versioned buckets differently from non-versioned ones?**
**A:** In non-versioned buckets, expiration permanently deletes the object. In versioned buckets, expiration of the *current version* creates a delete marker. You must specifically configure `NoncurrentVersionExpiration` to delete older versions of an object.

**Q: Can you use S3 Lifecycle to move objects from S3 Glacier back to S3 Standard?**
**A:** No. S3 Lifecycle transitions are designed to move data to lower-cost, less-frequently-accessed storage classes. Moving data back to a higher-tier class (like Standard) must be done manually via a Restore or Copy operation.

**Q: What is the \"AbortIncompleteMultipartUpload\" action?**
**A:** It is a lifecycle action that automatically deletes the parts of a multipart upload that did not complete within a specified number of days. This prevents you from being charged for storage of partial files that are no longer being uploaded.

**Q: Does a Lifecycle policy apply to existing objects or only new ones?**
**A:** It applies to **both**. When you add or update a lifecycle configuration, S3 scans existing objects and applies the rules to those that already meet the criteria (e.g., objects older than 30 days).
