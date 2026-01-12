---
---

# Cloud Providers (AWS, Azure, GCP)

In the modern software architecture landscape, the choice of cloud provider is a strategic decision that influences the technology stack, operational costs, and scalability of an application. The "Big Three"—Amazon Web Services (AWS), Microsoft Azure, and Google Cloud Platform (GCP)—dominate the market, each offering a unique value proposition.

## 1. High-level Comparison

Each provider has a distinct "philosophy" and market focus:

| Provider | Market Philosophy | Best For |
| :--- | :--- | :--- |
| **AWS (Amazon)** | **The Builder's Cloud**. Prioritizes breadth and depth of services. If it exists in the cloud, AWS has a service for it. First-mover advantage. | Startups, tech-heavy enterprises, and projects requiring specialized services (e.g., Satellite, Quantum). |
| **Azure (Microsoft)** | **The Enterprise's Cloud**. Deep integration with the Microsoft ecosystem (Windows, Office 365, Active Directory). Strong focus on hybrid cloud. | Large enterprises with existing Microsoft licenses and heavy reliance on .NET/Windows Server. |
| **GCP (Google)** | **The Data & AI Cloud**. Built on the same infrastructure that powers Google. Known for industry-leading Kubernetes (GKE), big data, and machine learning. | Data-centric projects, high-performance computing, and containerized microservices. |

## 2. Service Mapping

Architects must be able to translate requirements across different providers. Below is a mapping of core services:

| Category | AWS | Azure | GCP |
| :--- | :--- | :--- | :--- |
| **Compute (VMs)** | EC2 (Elastic Compute Cloud) | Virtual Machines | Compute Engine |
| **Storage (Object)** | S3 (Simple Storage Service) | Blob Storage | Cloud Storage |
| **Database (Relational)** | RDS (Relational Database Service) | Azure SQL Database | Cloud SQL |
| **Kubernetes** | EKS (Elastic Kubernetes Service) | AKS (Azure Kubernetes Service) | GKE (Google Kubernetes Engine) |
| **Serverless (Functions)** | Lambda | Azure Functions | Cloud Functions |
| **Networking (VPC)** | VPC (Virtual Private Cloud) | VNet (Virtual Network) | VPC (Virtual Private Cloud) |
| **IAM** | AWS IAM | Microsoft Entra ID (fka Azure AD) | Cloud IAM |

## 3. Architect's Role: Multi-Cloud vs. Hybrid Cloud

A Software Architect must decide between different cloud strategies based on business needs, risk tolerance, and technical complexity.

### Multi-Cloud Strategy
*   **Definition**: Using multiple public cloud providers for different services or to avoid vendor lock-in.
*   **Pros**: Increased reliability (availability across providers), bargaining power with vendors, and the ability to choose "best-of-breed" services (e.g., AWS for compute, GCP for AI).
*   **Cons**: Significant operational complexity, higher training costs, and difficult data egress costs.
*   **Architect's Note**: True "cloud-agnostic" architecture is expensive. Often, it is better to use one primary provider and only use a second for specific high-value features.

### Hybrid Cloud Strategy
*   **Definition**: Combining on-premises infrastructure (private cloud) with public cloud services.
*   **Pros**: Regulatory compliance (data kept on-prem), gradual migration path, and leveraging existing hardware investments.
*   **Cons**: Complex networking (Direct Connect/ExpressRoute) and potential latency between on-prem and cloud.
*   **Architect's Note**: This is the default for most large enterprises transitioning to the cloud. Consistency in tooling (e.g., OpenStack, Anthos, or Azure Stack) is key.

---

## 4. Go Implementation: AWS SDK for Go (v2)

Modern cloud interactions are often automated via SDKs. The **AWS SDK for Go v2** is the current standard for interacting with AWS services programmatically.

### Listing S3 Buckets
This example demonstrates how to initialize the SDK and list all S3 buckets in an account.

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
	// 1. Load the SDK's default configuration (from environment variables, shared config files, etc.)
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	// 2. Create an Amazon S3 service client
	client := s3.NewFromConfig(cfg)

	// 3. Build the request and call the ListBuckets operation
	output, err := client.ListBuckets(context.TODO(), &s3.ListBucketsInput{})
	if err != nil {
		log.Fatalf("failed to list buckets, %v", err)
	}

	fmt.Println("Your S3 Buckets:")
	for _, bucket := range output.Buckets {
		// Note: Most fields in the SDK v2 are pointers
		fmt.Printf("* %s (Created: %v)\n", *bucket.Name, *bucket.CreationDate)
	}
}
```

---

## 5. Interview Preparation

### Standard Interview Questions

**Q1: What are the main factors to consider when choosing a cloud provider for a new architecture?**
*   **Answer**: Factors include:
    *   **Existing Ecosystem**: Does the team already have expertise in one provider? (e.g., .NET shop -> Azure).
    *   **Service Catalog**: Does the provider offer the specific high-level services needed (e.g., specific AI models, managed DBs)?
    *   **Compliance & Data Sovereignty**: Does the provider have regions in the required legal jurisdictions?
    *   **Cost**: Comparison of Reserved Instances, Savings Plans, and data egress costs.
    *   **Reliability**: SLAs and historical uptime of specific regions.

**Q2: Explain the concept of "Shared Responsibility Model".**
*   **Answer**: The cloud provider is responsible for the **security OF the cloud** (infrastructure, hardware, managed services). The customer is responsible for **security IN the cloud** (data protection, IAM configuration, OS patching for VMs, network firewall rules).

**Q3: Why would an architect choose GKE (Google Kubernetes Engine) over EKS (AWS) or AKS (Azure)?**
*   **Answer**: GKE is often considered the most mature managed Kubernetes service because Kubernetes was originally developed by Google. It offers superior auto-scaling capabilities (Cluster Autoscaler, Node Auto-Provisioning), seamless integration with GCP's global VPC, and "Autopilot" mode for managed operations.

**Q4: How do you handle vendor lock-in as a Software Architect?**
*   **Answer**: Lock-in can be mitigated at several levels:
    *   **Infrastructure**: Use Infrastructure as Code (Terraform) to define resources (though providers' resources are different).
    *   **Application**: Use containerization (Docker/Kubernetes) to make the application portable.
    *   **Code**: Use the Strategy pattern or hexagonal architecture to abstract cloud-specific SDKs behind interfaces.
    *   **Data**: Use open-standard databases (PostgreSQL, MySQL) rather than proprietary ones (DynamoDB, CosmosDB), though this may sacrifice some managed benefits.

**Q5: What is the difference between S3 (Object Storage) and EBS (Block Storage)?**
*   **Answer**: 
    *   **EBS** is block storage designed for use with a single EC2 instance (like a hard drive). It's used for OS boot volumes and databases.
    *   **S3** is object storage accessible via HTTP API. It is designed for massive scale, durable storage of unstructured data (images, logs, backups), and can be accessed globally.
