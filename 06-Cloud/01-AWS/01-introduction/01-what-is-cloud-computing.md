---
tags: ['aws', 'roadmap']
---

## Summary
Cloud computing is a model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction. According to the **NIST (National Institute of Standards and Technology) Special Publication 800-145**, cloud computing is defined by five essential characteristics, three service models (IaaS, PaaS, SaaS), and four deployment models (Public, Private, Hybrid, Community).

## Detailed Explanation

### The 5 Essential Characteristics of Cloud Computing

1. **On-demand Self-service**
   - **Definition**: A consumer can unilaterally provision computing capabilities, such as server time and network storage, as needed automatically without requiring human interaction with each service’s provider.
   - **Key Benefit**: Speed and agility. No need to wait for a hardware technician to rack a server.
   - **AWS Context**: Creating an EC2 instance or an S3 bucket via the AWS Console, CLI, or SDK happens instantly.

2. **Broad Network Access**
   - **Definition**: Capabilities are available over the network and accessed through standard mechanisms that promote use by heterogeneous thin or thick client platforms (e.g., mobile phones, tablets, laptops, and workstations).
   - **Key Benefit**: Accessibility from anywhere with an internet connection.
   - **AWS Context**: AWS services are accessible via public endpoints over HTTPS, or privately via VPC endpoints and Direct Connect.

3. **Resource Pooling**
   - **Definition**: The provider’s computing resources are pooled to serve multiple consumers using a multi-tenant model, with different physical and virtual resources dynamically assigned and reassigned according to consumer demand.
   - **Key Benefit**: Economies of scale. High utilization of underlying hardware reduces costs for everyone.
   - **AWS Context**: Multiple customers share the same physical server (isolated by the Nitro System or Xen/KVM hypervisors). Users generally don't know the exact physical location of the hardware.

4. **Rapid Elasticity**
   - **Definition**: Capabilities can be elastically provisioned and released, in some cases automatically, to scale rapidly outward and inward commensurate with demand.
   - **Key Benefit**: Scalability. The cloud appears infinite to the user; you can scale from 1 to 10,000 servers and back down in minutes.
   - **AWS Context**: **Amazon EC2 Auto Scaling** automatically adds or removes instances based on demand (CPU, traffic, etc.).

5. **Measured Service**
   - **Definition**: Cloud systems automatically control and optimize resource use by leveraging a metering capability at some level of abstraction appropriate to the type of service (e.g., storage, processing, bandwidth, and active user accounts).
   - **Key Benefit**: Pay-as-you-go. You only pay for what you use, turning capital expenditure (CapEx) into operating expenditure (OpEx).
   - **AWS Context**: **AWS Billing** provides detailed reports on usage (e.g., per-second billing for EC2, per-GB for S3).

---

### Go Example: Programmatic Resource Access (AWS SDK v2)

In Go, interacting with the cloud starts with the AWS SDK. This demonstrates **On-demand self-service** (programmatic provisioning) and **Broad network access** (connecting via standard protocols).

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/s3"
)

func main() {
	// Initialize the SDK configuration (On-demand Self-service)
	// This automatically searches for credentials in ENV, ~/.aws/credentials, or IAM Roles.
	ctx := context.TODO()
	cfg, err := config.LoadDefaultConfig(ctx, config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	// Create an S3 client
	// S3 is a globally distributed service (Resource Pooling)
	client := s3.NewFromConfig(cfg)

	// List buckets to verify connectivity (Broad Network Access)
	output, err := client.ListBuckets(ctx, &s3.ListBucketsInput{})
	if err != nil {
		log.Fatalf("unable to list buckets, %v", err)
	}

	fmt.Println("Found buckets in your account:")
	for _, bucket := range output.Buckets {
		fmt.Printf("- %s (Created: %v)\n", *bucket.Name, bucket.CreationDate)
	}
}
```

## Interview Questions

*   **Q: What is the NIST definition of Cloud Computing?**
    *   **A:** NIST defines it as a model for enabling convenient, on-demand network access to a shared pool of configurable computing resources that can be rapidly provisioned and released with minimal management effort.

*   **Q: Explain the difference between Scalability and Elasticity.**
    *   **A:** **Scalability** is the ability to handle growth (adding more resources). **Elasticity** is the ability to automatically grow *and shrink* based on demand. Elasticity is a subset of scalability that focuses on automation and cost-efficiency.

*   **Q: What is "Multi-tenancy" in the context of Resource Pooling?**
    *   **A:** Multi-tenancy is an architecture where a single instance of software (or hardware) serves multiple customers (tenants). Resources are logically isolated but physically shared to maximize efficiency.

*   **Q: How does the "Measured Service" characteristic impact business finances?**
    *   **A:** It allows businesses to shift from **CapEx** (high upfront investment in hardware) to **OpEx** (monthly utility-style bills), improving cash flow and reducing financial risk.

*   **Q: Why is "Broad Network Access" important for modern applications?**
    *   **A:** It ensures that services are reachable from any device (IoT, Mobile, Web) using standard internet protocols (HTTP/HTTPS), enabling the "anywhere, anytime" nature of cloud services.
