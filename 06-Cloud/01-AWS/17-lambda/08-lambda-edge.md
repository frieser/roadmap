#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'lambda']
---

## Summary
Lambda@Edge is an extension of AWS Lambda that allows you to run functions at AWS Edge locations, closer to your users. It integrates with Amazon CloudFront to intercept and modify requests and responses at four specific stages of the delivery process. This enables low-latency customization of content, such as A/B testing, header manipulation, and personalized delivery, without the overhead of reaching the origin server for every request.

## Detailed Explanation
**How Lambda@Edge Works**
Lambda@Edge functions are triggered by Amazon CloudFront events. When you associate a function with a CloudFront distribution, CloudFront replicates the code to edge locations globally.

### **CloudFront Triggers (The Four Events)**
1.  **Viewer Request**: Triggered when CloudFront receives a request from a viewer (before checking the cache). Ideal for URL rewrites, redirects, and authentication.
2.  **Origin Request**: Triggered only on a cache miss, before CloudFront forwards the request to the origin. Useful for modifying the request to the origin (e.g., adding API keys or custom headers).
3.  **Origin Response**: Triggered after CloudFront receives a response from the origin but before it caches the object. Used for manipulating headers from the origin (e.g., security headers like HSTS or CSP).
4.  **Viewer Response**: Triggered before CloudFront sends the response to the viewer (regardless of cache status). Used for final response transformations.

### **Key Use Cases**
- **Dynamic Content Selection**: Serving different assets based on the `User-Agent` or `CloudFront-Is-Mobile-Viewer` headers.
- **Security & Authorization**: Validating JWT tokens at the edge to block unauthorized traffic before it hits the origin.
- **A/B Testing**: Implementing experiment logic by modifying the request path to point to different origin folders.
- **SEO & Social Media Optimization**: Serving pre-rendered content to crawlers while serving the standard application to users.

### **Limitations & Restrictions**
- **Region**: Functions must be created in the `us-east-1` (N. Virginia) region.
- **Runtimes**: Supports only **Node.js** and **Python**. (Go is not natively supported for Lambda@Edge).
- **Execution Limits**: Viewer triggers have stricter quotas (128MB RAM, 5s timeout) than Origin triggers (up to 10GB RAM, 30s timeout).
- **Environment Variables**: Not supported. Configuration must be hardcoded, bundled, or fetched from SSM/Secrets Manager.

### **Lambda@Edge vs. CloudFront Functions**
| Feature | CloudFront Functions | Lambda@Edge |
| --- | --- | --- |
| **Runtime** | JavaScript (subset) | Node.js, Python |
| **Execution Time** | < 1ms | Up to 5s (Viewer) / 30s (Origin) |
| **Scale** | 10M+ RPS | Lower than CF Functions |
| **Network Access** | No | Yes |
| **Triggers** | Viewer Request/Response | All 4 triggers |

### **Go Application Context**
While Lambda@Edge doesn't support a Go runtime, Go is commonly used to manage the infrastructure. Below is a conceptual example using the AWS SDK for Go to associate a Lambda@Edge function with a CloudFront distribution.

```go
package main

import (
	"context"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/cloudfront"
	"github.com/aws/aws-sdk-go-v2/service/cloudfront/types"
	"log"
)

func main() {
	cfg, _ := config.LoadDefaultConfig(context.TODO())
	client := cloudfront.NewFromConfig(cfg)

	distId := "EXAMPLE_DIST_ID"
	// Lambda@Edge requires a specific versioned ARN (cannot use $LATEST)
	lambdaArn := "arn:aws:lambda:us-east-1:123456789012:function:my-edge-func:1"

	_, err := client.UpdateDistribution(context.TODO(), &cloudfront.UpdateDistributionInput{
		Id: &distId,
		DistributionConfig: &types.DistributionConfig{
			DefaultCacheBehavior: &types.DefaultCacheBehavior{
				LambdaFunctionAssociations: &types.LambdaFunctionList{
					Quantity: 1,
					Items: []types.LambdaFunctionAssociation{
						{
							EventType:         types.EventTypeViewerRequest,
							LambdaFunctionARN: &lambdaArn,
						},
					},
				},
				// ... other mandatory fields like TargetOriginId, ForwardedValues, etc.
			},
			// ...
		},
	})
	if err != nil {
		log.Fatalf("failed to update distribution: %v", err)
	}
}
```

## Interview Questions
1. **Q: What is the main difference between Viewer Request and Origin Request triggers?**
   **A:** Viewer Request triggers run for every request from a client, regardless of cache status. Origin Request triggers only run on a cache miss, right before CloudFront fetches content from the origin.

2. **Q: Why must Lambda@Edge functions be created in the us-east-1 region?**
   **A:** This is an AWS architectural requirement; the Lambda@Edge service uses the N. Virginia region as the global control plane to replicate function code to all edge locations.

3. **Q: Can you use environment variables in Lambda@Edge?**
   **A:** No, environment variables are not supported. You must hardcode values, bundle a config file, or use AWS Systems Manager (SSM) Parameter Store to fetch settings at runtime.

4. **Q: When would you choose CloudFront Functions over Lambda@Edge?**
   **A:** Choose CloudFront Functions for high-scale, simple tasks like header manipulation or URL rewrites that require sub-millisecond latency. Use Lambda@Edge for complex logic, origin triggers, or if you need to make external API calls.
