---
---

## Summary
Serverless architecture is a cloud-native development model that allows developers to build and run applications without managing the underlying infrastructure. It shifts the responsibility of server management, scaling, and availability to the cloud provider. From an architectural standpoint, it is characterized by **FaaS** (Function as a Service) for compute and **BaaS** (Backend as a Service) for integrated logic and storage.

## 1. FaaS vs BaaS
Serverless is the combination of these two paradigms:

| Feature | FaaS (Function as a Service) | BaaS (Backend as a Service) |
| :--- | :--- | :--- |
| **Definition** | Ephemeral, stateless compute units executed in response to events. | Third-party services that provide specific backend features via API. |
| **Examples** | AWS Lambda, Google Cloud Functions, Azure Functions. | Firebase Auth, Auth0, DynamoDB, AWS S3, Stripe. |
| **Logic** | Custom business logic written by the developer. | Pre-built logic managed by a provider. |
| **Lifetime** | Short-lived (seconds to minutes). | Persistent/Always available via API. |

**Architectural Insight**: FaaS provides the "glue" code that connects various BaaS components to form a complete application.

## 2. Event-Driven Nature
Serverless is inherently event-driven. Functions do not "listen" for requests; they are "pushed" into execution by triggers.

*   **Synchronous Triggers**: API Gateway (HTTP), direct SDK calls. The caller waits for a response.
*   **Asynchronous Triggers**: SQS (Queueing), SNS (Pub/Sub), EventBridge. The caller moves on; the platform handles retries.
*   **Stream-Based Triggers**: Kinesis, DynamoDB Streams. Functions process batches of records from a continuous stream.

### Architectural Patterns:
*   **Choreography**: Services communicate via events without a central coordinator (highly decoupled).
*   **Orchestration**: A central controller (e.g., AWS Step Functions) manages the flow between multiple serverless functions (better for complex workflows).

## 3. Cold Starts and Performance Trade-offs
A **Cold Start** occurs when the provider must initialize a new execution environment for a function.

### Execution Lifecycle:
1.  **Download code**: Pulling code from storage.
2.  **Start container**: Provisioning the runtime environment.
3.  **Init runtime**: Bootstrapping the language runtime (e.g., JVM, Node.js, Go).
4.  **Execute handler**: Running the actual logic (Warm Start starts here).

### Mitigation Strategies:
*   **Provisioned Concurrency**: Keeping a set number of environments "warm" (increases cost).
*   **Language Choice**: Compiled languages like Go and Rust have significantly faster init phases than Java or Python.
*   **Memory Tuning**: Increasing memory often provides a proportional increase in CPU, speeding up initialization.
*   **Minimizing Package Size**: Smaller binaries/packages reduce the "Download code" phase.

## 4. Vendor Lock-in Considerations
Serverless architectures are deeply integrated with proprietary vendor ecosystems (triggers, IAM, managed services).

### Lock-in Vectors:
*   **Proprietary APIs**: Relying on specific BaaS features (e.g., DynamoDB's unique API).
*   **Event Schemas**: Each cloud provider has different formats for events (S3 vs GCS).
*   **Deployment Tooling**: SAM, CloudFormation, or Terraform configurations specific to the provider.

### Portability Strategies:
*   **Hexagonal Architecture**: Keep business logic in "pure" modules, using adapters for vendor-specific inputs (triggers) and outputs (BaaS).
*   **Standardized Frameworks**: Use tools like **Serverless Framework** or **SST** to abstract infrastructure definitions.
*   **CloudEvents**: Use the CNCF CloudEvents standard to normalize event schemas across platforms.

## 5. Go-specific Context
Go is a premier choice for serverless due to:
*   **Fast cold starts**: Native binaries require no VM/Runtime warmup.
*   **Resource efficiency**: Low memory footprint allows for lower cost and higher density.
*   **Static typing**: Robustness for distributed, event-driven systems.

### Example: AWS Lambda in Go
```go
package main

import (
	"context"
	"github.com/aws/aws-lambda-go/lambda"
)

func HandleRequest(ctx context.Context, name string) (string, error) {
	return "Hello " + name, nil
}

func main() {
	lambda.Start(HandleRequest)
}
```

## Interview Questions
**Q: When should you NOT use Serverless?**
**A:** For long-running processes (exceeding 15 mins), steady-state high-traffic workloads (where reserved instances are cheaper), or applications requiring very low sub-millisecond latency (due to cold start risks).

**Q: How do you manage state in FaaS?**
**A:** FaaS is stateless. State must be externalized to BaaS like Redis (ElastiCache), DynamoDB, or S3.
