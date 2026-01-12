---
---

# Amazon Web Services (AWS)

AWS is the market leader in cloud computing, offering over 200 fully featured services. For DevOps engineers, it provides the most comprehensive set of IaaS (Infrastructure as Service) primitives for building scalable, fault-tolerant architectures.

## Summary

AWS offers a vast ecosystem of services. The core building blocks for almost any architecture are **EC2** (Compute), **S3** (Storage), **RDS** (Database), and **VPC** (Networking). AWS pioneered the "Shared Responsibility Model" for security and offers granular access control via **IAM**. Its Go SDK (`aws-sdk-go-v2`) is modular and robust, making it a favorite for building custom DevOps tools (like Terraform providers or CLI utilities).

## Detailed Explanation

### 1. Core Services
*   **EC2 (Elastic Compute Cloud)**: Virtual servers. You choose the OS, CPU/RAM, and networking.
*   **S3 (Simple Storage Service)**: Object storage with 99.999999999% (11 9s) durability. Used for backups, static site hosting, and data lakes.
*   **RDS (Relational Database Service)**: Managed SQL databases (Postgres, MySQL). Handles backups, patching, and scaling.
*   **VPC (Virtual Private Cloud)**: Your isolated network environment. You define subnets, route tables, and gateways.

### 2. DevOps on AWS
*   **Infrastructure as Code**: CloudFormation (native) or Terraform (standard).
*   **CI/CD**: CodePipeline, CodeBuild, CodeDeploy.
*   **Monitoring**: CloudWatch (Metrics & Logs).

---

## Go Implementation Example

The AWS SDK for Go V2 is modular. You import only the services you need.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/ec2"
)

func main() {
	// 1. Load configuration (creds from env vars, profile, or IAM role)
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	// 2. Create EC2 Client
	client := ec2.NewFromConfig(cfg)

	// 3. Describe Instances
	output, err := client.DescribeInstances(context.TODO(), &ec2.DescribeInstancesInput{})
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("EC2 Instances:")
	for _, reservation := range output.Reservations {
		for _, instance := range reservation.Instances {
			fmt.Printf("- ID: %s | Type: %s | State: %s\n",
				*instance.InstanceId,
				instance.InstanceType,
				instance.State.Name,
			)
		}
	}
}
```

## Interview Questions

**Q: What is the difference between a Security Group and a Network ACL?**
**A:**
*   **Security Group**: Acts as a **stateful** firewall at the *instance* level. If you allow inbound traffic on port 80, the return traffic is automatically allowed.
*   **NACL (Network Access Control List)**: Acts as a **stateless** firewall at the *subnet* level. You must explicitly allow both inbound and outbound traffic.

**Q: Explain the concept of an Availability Zone (AZ) vs. a Region.**
**A:** A **Region** is a separate geographic area (e.g., `us-east-1` in N. Virginia). Each Region has multiple, isolated locations known as **Availability Zones** (e.g., `us-east-1a`). AZs are connected with low latency links. To achieve high availability, you deploy applications across multiple AZs within a Region.

**Q: How does IAM Roles differ from IAM Users?**
**A:** An **IAM User** represents a person or service with permanent long-term credentials (password/access keys). An **IAM Role** is an identity with specific permissions that can be *assumed* by a user, service (like EC2 or Lambda), or external identity for a temporary session. DevOps best practice is to use Roles for services to avoid managing long-lived keys.
