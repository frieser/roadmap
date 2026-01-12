#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'cloudfront']
---

## Summary
Amazon CloudFront distributions are the primary configuration unit for content delivery. A distribution links one or more **Origins** (where the original content resides) to the **AWS Edge Network**. Historically, AWS provided both **Web** and **RTMP** distributions. However, RTMP was deprecated in late 2020, making Web distributions the universal standard for delivering static assets, dynamic web content, and modern video streaming (HLS/DASH).

## Detailed Explanation

### Origins: S3 vs. Custom (ALB/EC2)
A distribution can have multiple origins, which are defined by their DNS domain name and protocol:
- **Amazon S3**: Used for static assets (images, JS, CSS). Best practices include using **Origin Access Control (OAC)** to ensure users can only access the S3 bucket via CloudFront, preventing direct access and bypass of security/caching logic.
- **Custom Origins (ALB/EC2/External)**: Used for dynamic content. CloudFront optimizes the path from the edge to these origins using the AWS global network backbone, reducing latency even for non-cacheable content.

### Web vs. RTMP Distributions
- **Web Distributions**: Support HTTP/HTTPS protocols. They are used for all modern web use cases, including static files, dynamic content delivery, and media streaming protocols like **HLS** (HTTP Live Streaming) and **MPEG-DASH**.
- **RTMP Distributions (Deprecated)**: These were used for streaming media using the Adobe Real-Time Messaging Protocol (Flash). **AWS discontinued support for RTMP on December 31, 2020**. Legacy streaming use cases must now migrate to Web distributions using HLS/DASH.

### Caching Behavior and Policies
CloudFront provides granular control over what gets cached and for how long:
- **Cache Key**: The combination of values (URL, query strings, headers, cookies) that uniquely identifies a file in the cache. To maximize the **Cache Hit Ratio**, you should keep the cache key as simple as possible.
- **Cache Policy**: Defines the TTL (Minimum, Maximum, and Default) and specifies which parameters are included in the cache key.
- **Origin Request Policy**: Allows you to forward specific headers, cookies, or query strings to the origin *without* making them part of the cache key. This is a crucial optimization to avoid cache fragmentation.
- **TTL (Time To Live)**: Controlled by the origin's `Cache-Control` or `Expires` headers. If these are missing, CloudFront uses the default values defined in the distribution.

### Cache Invalidation
When content is updated at the origin, the edge locations will still serve the old version until the TTL expires. An **Invalidation** request forces the cache to clear specified paths (e.g., `/images/*` or `/index.html`), forcing CloudFront to fetch the latest version from the origin on the next request.

### Go Application: Programmatic Invalidation
In professional DevOps workflows, you often need to invalidate the cache programmatically after a CI/CD deployment. Below is a Go example using the **AWS SDK for Go v2**.

```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/cloudfront"
	"github.com/aws/aws-sdk-go-v2/service/cloudfront/types"
)

// InvalidateCloudFrontCache creates an invalidation batch for the specified paths
func InvalidateCloudFrontCache(distID string, paths []string) {
	ctx := context.TODO()
	
	// Load AWS configuration (using default credentials chain)
	cfg, err := config.LoadDefaultConfig(ctx)
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	// Create CloudFront client
	client := cloudfront.NewFromConfig(cfg)

	// A unique reference for the invalidation request
	callerReference := fmt.Sprintf("invalidation-%d", time.Now().Unix())

	input := &cloudfront.CreateInvalidationInput{
		DistributionId: aws.String(distID),
		InvalidationBatch: &types.InvalidationBatch{
			CallerReference: aws.String(callerReference),
			Paths: &types.Paths{
				Quantity: aws.Int32(int32(len(paths))),
				Items:    paths,
			},
		},
	}

	// Execute invalidation
	output, err := client.CreateInvalidation(ctx, input)
	if err != nil {
		log.Fatalf("failed to create invalidation: %v", err)
	}

	fmt.Printf("Successfully created invalidation. ID: %s\n", *output.Invalidation.Id)
}

func main() {
	distributionID := "E1234567890ABC"
	targetPaths := []string{"/index.html", "/static/css/*"}
	InvalidateCloudFrontCache(distributionID, targetPaths)
}
```

## Interview Questions

**Q: What is the current status of RTMP distributions in AWS CloudFront?**
**A:** RTMP distributions are deprecated and have been unsupported since December 31, 2020. They were used for legacy Flash media streaming. All modern streaming (HLS/DASH) is now delivered via standard Web distributions.

**Q: How do you protect an S3 bucket origin from being accessed directly?**
**A:** You should use **Origin Access Control (OAC)**. OAC allows CloudFront to sign requests to S3 using AWS Signature Version 4. You then configure the S3 bucket policy to only allow the specific CloudFront distribution's service principal (`cloudfront.amazonaws.com`) to perform `s3:GetObject` actions.

**Q: What is the difference between a Cache Policy and an Origin Request Policy?**
**A:** A **Cache Policy** determines which parameters (headers, cookies, query strings) are used to create the **Cache Key** (and thus affect cache hits). An **Origin Request Policy** determines which parameters are forwarded to the origin but does *not* include them in the cache key, which helps maintain a high cache hit ratio.

**Q: How can you update a file in CloudFront without waiting for the TTL to expire or paying for an invalidation?**
**A:** The recommended approach is **Object Versioning**. By including a version or hash in the filename (e.g., `app.v2.js` instead of `app.js`), you create a completely new cache key. This ensures immediate updates for users and avoids the costs and propagation delays associated with invalidation requests.

**Q: When would you use a TTL of 0 in a CloudFront distribution?**
**A:** A TTL of 0 is used for dynamic content that must be fetched from the origin every time but still benefits from CloudFront's optimized network routing to the origin (reducing latency over the public internet) and security features like AWS WAF integration.
