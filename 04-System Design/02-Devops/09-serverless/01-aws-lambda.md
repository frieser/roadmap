---
---

# AWS Lambda

AWS Lambda is the pioneer of modern Function-as-a-Service (FaaS) computing. It allows you to run code without provisioning or managing servers. For DevOps, Lambda is ubiquitous: it powers everything from REST APIs to automated infrastructure cleanup scripts.

## Summary

Lambda runs your code in response to **events** (Triggers). You pay only for the compute time you consume (millisconds). Key concepts include **Cold Starts** (latency when a new execution environment is spun up) and **Layers** (shared libraries). Go is a first-class citizen in Lambda, offering extremely fast startup times compared to Java or .NET.

## Detailed Explanation

### 1. Core Concepts
*   **Triggers**: Events that start the function. Common triggers:
    *   **API Gateway**: HTTP requests.
    *   **S3**: File uploads.
    *   **DynamoDB Streams**: Database changes.
    *   **EventBridge (CloudWatch Events)**: Cron jobs or system state changes.
*   **Cold Starts**: When a function hasn't run recently, AWS must initialize a new micro-VM (Firecracker). This adds latency. Go binaries are static and small, minimizing this.
*   **Layers**: A way to package libraries or custom runtimes separately from your function code, keeping your deployment artifact small.

### 2. DevOps with Lambda
*   **IaC**: Defined via SAM (Serverless Application Model), CloudFormation, or Terraform.
*   **Logging**: Automatically sends stdout/stderr to CloudWatch Logs.

---

## Go Implementation Example

Using the `github.com/aws/aws-lambda-go/lambda` library.

### 1. The Handler Code (`main.go`)
```go
package main

import (
	"context"
	"fmt"

	"github.com/aws/aws-lambda-go/lambda"
)

// Event struct (Custom or predefined like events.APIGatewayProxyRequest)
type MyEvent struct {
	Name string `json:"name"`
}

type MyResponse struct {
	Message string `json:"message"`
}

// HandleRequest is the main entry point
func HandleRequest(ctx context.Context, event MyEvent) (MyResponse, error) {
	return MyResponse{
		Message: fmt.Sprintf("Hello %s from Go Lambda!", event.Name),
	}, nil
}

func main() {
	// Start the handler
	lambda.Start(HandleRequest)
}
```

### 2. Building for AWS
Since Lambda runs on Linux, you must compile for it:
```bash
GOOS=linux GOARCH=amd64 go build -o bootstrap main.go
zip function.zip bootstrap
```
*Note: For `provided.al2` runtime, the binary must be named `bootstrap`.*

## Interview Questions

**Q: What is a "Cold Start" and how can you mitigate it in AWS Lambda?**
**A:** A Cold Start occurs when AWS has to provision a new execution environment for your function because no idle instances are available. Mitigations include:
1.  **Language Choice**: Use languages with fast startup (Go, Rust, Node.js) over Java/C#.
2.  **Provisioned Concurrency**: Pay to keep a certain number of instances "warm" and ready.
3.  **Minimize Package Size**: Smaller binaries download and unpack faster.

**Q: What is the "execution context" reuse?**
**A:** After a Lambda function runs, AWS freezes the environment. If another request comes in quickly, AWS "thaws" it and reuses it. Global variables (like database connections) initialized *outside* the handler function persist. DevOps engineers use this to cache DB connections or secrets, significantly improving performance on subsequent runs.

**Q: Explain the difference between `zip` deployment and `container` deployment for Lambda.**
**A:**
*   **Zip**: Standard. You upload a zip file containing your code. Limited to 250MB unzipped. Faster cold starts typically.
*   **Container Image**: You package your code as a Docker image (up to 10GB). Allows using complex dependencies (like ffmpeg or ML models) that don't fit in a zip. AWS caches the image layers to optimize startup.
