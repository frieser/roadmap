---
---

# AWS ECR (Elastic Container Registry)

Amazon Elastic Container Registry (ECR) is a fully managed container registry that makes it easy to store, manage, share, and deploy container images and artifacts.

## Summary

ECR is the standard choice for AWS-native workloads (ECS, EKS, Lambda). It is highly available, secure (IAM integration), and supports **Image Scanning** (for CVEs) and **Lifecycle Policies** (auto-cleanup).

## Detailed Explanation

### 1. Key Concepts
*   **Repositories**: A place to store Docker images. One repository per image name (e.g., `my-app`, `redis-custom`).
*   **Authorization Token**: To push/pull, you must authenticate. The token is valid for 12 hours. `aws ecr get-login-password | docker login ...`
*   **Immutable Tags**: A setting to prevent overwriting image tags. Critical for production stability.
*   **Lifecycle Policies**: Rules to expire old images (e.g., "Keep only the last 10 untagged images").

### 2. Integration
ECR is deeply integrated with **ECS** (Elastic Container Service) and **EKS** (Kubernetes). IAM roles allow EC2 instances or Fargate tasks to pull images without managing explicit Docker credentials.

---

## Go Implementation Example

Using `aws-sdk-go-v2` to interact with ECR. Common tasks include checking if a repo exists or getting an auth token.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/ecr"
)

func main() {
	cfg, err := config.LoadDefaultConfig(context.TODO())
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := ecr.NewFromConfig(cfg)

	// 1. List Repositories
	output, err := client.DescribeRepositories(context.TODO(), &ecr.DescribeRepositoriesInput{})
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("ECR Repositories:")
	for _, repo := range output.Repositories {
		fmt.Printf("- %s (URI: %s)\n", *repo.RepositoryName, *repo.RepositoryUri)
	}

	// 2. Get Authorization Token (for Docker Login)
	authOutput, err := client.GetAuthorizationToken(context.TODO(), &ecr.GetAuthorizationTokenInput{})
	if err != nil {
		log.Fatal(err)
	}

	for _, authData := range authOutput.AuthorizationData {
		fmt.Printf("Token Endpoint: %s\n", *authData.ProxyEndpoint)
		// Token is base64 encoded user:password
		// fmt.Println(*authData.AuthorizationToken) 
	}
}
```

## Interview Questions

**Q: How does ECR Cross-Region Replication work?**
**A:** ECR allows you to configure replication rules. When you push an image to a repository in the source region (e.g., `us-east-1`), ECR automatically copies it to configured destination regions (e.g., `eu-west-1`). This is essential for low-latency global deployments and disaster recovery.

**Q: What is "Image Scanning" in ECR?**
**A:** ECR can scan your Docker images for Common Vulnerabilities and Exposures (CVEs) in the OS packages. You can configure "Scan on Push" to automatically check every upload. The results are available via the console or API, and high-severity findings can block deployment pipelines.

**Q: Why use ECR over Docker Hub?**
**A:**
1.  **Security**: ECR uses IAM for access control, eliminating the need to manage static Docker Hub credentials.
2.  **Performance**: Pulling images from ECR to AWS infrastructure (EC2/ECS) is faster and doesn't traverse the public internet (if VPC endpoints are used).
3.  **Cost**: Data transfer within the same region is often cheaper or free.
