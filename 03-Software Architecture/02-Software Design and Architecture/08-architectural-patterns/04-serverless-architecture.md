---
---

## Summary
**Serverless Architecture** refers to a cloud computing execution model where the cloud provider (AWS, Azure, GCP) manages the allocation and provisioning of servers. The developer simply writes code (functions) that run in response to events (HTTP requests, database changes, file uploads). It is characterized by **FaaS** (Function as a Service), **statelessness**, **ephemeral compute**, and **pay-per-use** pricing.

## Detailed Explanation

### 1. Core Concepts
*   **FaaS (Function as a Service)**: Code is deployed as individual functions (e.g., AWS Lambda).
*   **Event-Driven**: Functions sleep until an event triggers them (e.g., S3 upload, API Gateway hit).
*   **Stateless**: Functions are ephemeral. They start, run, and die. They cannot rely on local memory between executions; state must be stored externally (DB, Redis).
*   **Cold Starts**: The latency incurred when the provider spins up a new container to handle a request after a period of inactivity.

### 2. Benefits
*   **No Ops**: No server management/patching.
*   **Auto-scaling**: Scales from 0 to 1000s of concurrent requests automatically.
*   **Cost**: You pay only for the compute time used (milliseconds).

### 3. Drawbacks
*   **Cold Starts**: Initial request latency.
*   **Vendor Lock-in**: deeply tied to provider APIs (DynamoDB, SQS).
*   **Complexity**: Debugging distributed functions is harder than a monolith.

## Go Code Example

Go is a first-class citizen in the serverless world (especially AWS Lambda) due to its fast startup time and small binary size.

**Requirements**: `go get github.com/aws/aws-lambda-go/lambda`

```go
package main

import (
	"context"
	"fmt"
	"github.com/aws/aws-lambda-go/lambda"
)

// Event defines the expected JSON input
type MyEvent struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

// Response defines the JSON output
type MyResponse struct {
	Message string `json:"message"`
	Success bool   `json:"success"`
}

// HandleRequest is the main entry point (the "handler")
// Context provides runtime info (timeout, request ID)
func HandleRequest(ctx context.Context, event MyEvent) (MyResponse, error) {
	if event.Name == "" {
		return MyResponse{
			Message: "Error: Name is required",
			Success: false,
		}, fmt.Errorf("missing name")
	}

	message := fmt.Sprintf("Hello %s, you are %d years old!", event.Name, event.Age)
	
	// Return the response object
	return MyResponse{
		Message: message,
		Success: true,
	}, nil
}

func main() {
	// The lambda.Start method delegates execution to the handler
	lambda.Start(HandleRequest)
}
```

## Interview Questions

### Q: What is a "Cold Start" and how does Go compare to Java or Node.js?
**A:** A Cold Start is the delay when the cloud provider creates a new container to run your function. Go generally has **very fast cold starts** compared to Java (JVM startup) or Node.js (runtime init), because Go compiles to a small, native binary. This makes Go an excellent choice for latency-sensitive serverless APIs.

### Q: How do you manage state in a Serverless application?
**A:** You **cannot** store state in the application memory (variables/globals) because the container might be destroyed immediately after the function finishes. All state must be externalized to persistent storage services like **DynamoDB, Redis (ElastiCache), or S3**. The function should be purely functional (Input -> Process -> Output).

### Q: What is the "Vendor Lock-in" risk with Serverless?
**A:** Serverless applications often rely heavily on proprietary services (AWS Lambda triggers, API Gateway, DynamoDB streams). Moving such an application to another provider (e.g., Azure Functions) requires significant rewriting of the infrastructure code (Terraform/CloudFormation) and often the application logic itself to adapt to different event payloads and service integrations.
