#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ec2']
---

## Summary
An **Elastic IP address (EIP)** is a reserved, static public IPv4 address designed for dynamic cloud computing in AWS. Unlike standard public IP addresses that change when an instance is stopped or terminated, an EIP remains allocated to your AWS account until you explicitly release it. Its primary value lies in its ability to be rapidly remapped between instances, enabling highly available architectures and seamless failover strategies.

## Detailed Explanation

### Core Concepts
An Elastic IP address acts as a fixed entry point for your applications. It is particularly useful when you need a consistent IP address for DNS records, firewall allow-listing, or SSL/TLS certificate bindings.

- **Regional Scope**: EIPs are tied to a specific AWS Region and cannot be moved across regions.
- **Persistence**: The IP address stays with your account even if the associated resource is deleted, provided you don't manually release it.
- **Remapping**: You can disassociate an EIP from one instance and associate it with another almost instantly. This is the cornerstone of many disaster recovery and high-availability setups.

### Pricing (2024/2026 Update)
As of **February 1, 2024**, AWS standardized public IPv4 pricing to encourage IP conservation.
- **Standard Charge**: Every public IPv4 address, including Elastic IPs, costs **$0.005 per IP per hour**.
- **In-use vs. Idle**: Previously, one in-use EIP was free. Now, you pay the $0.005/hour rate regardless of whether the EIP is associated with a running instance or sitting idle in your account.
- **Unattached Costs**: Because you are charged $0.005/hour for any allocated IP, an unattached (idle) EIP costs roughly **$3.60 per month**. It is critical to release EIPs that are no longer needed to avoid unnecessary billing.
- **BYOIP Exception**: Public IPv4 addresses that you bring to AWS via **BYOIP (Bring Your Own IP)** are not charged by AWS.

### Association and Network Interfaces
EIPs can be associated with:
1. **EC2 Instances**: Directly mapped to the primary network interface.
2. **Network Interfaces (ENI)**: Can be attached to any ENI, providing flexibility for multi-homed instances or virtual appliances.
3. **NAT Gateways**: Required for NAT Gateways to communicate with the internet.
4. **Network Load Balancers**: Used to provide static IPs for load-balanced traffic.

### Reverse DNS and Security
AWS allows you to configure **Reverse DNS (rDNS)** for your Elastic IP addresses. This is critical for running mail servers (SMTP), as many receiving servers block mail from IPs without a valid pointer (PTR) record. You can manage this via the EC2 console or by contacting AWS support.

### Go Application: Managing Elastic IPs
In a Go-based automation script, you can automate failover by remapping an EIP from a failed instance to a healthy one using the AWS SDK v2.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/ec2"
	"github.com/aws/aws-sdk-go-v2/service/ec2/types"
	"github.com/aws/aws-sdk-go/aws"
)

func main() {
	ctx := context.TODO()
	// Load AWS configuration (default region from env or config file)
	cfg, err := config.LoadDefaultConfig(ctx, config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := ec2.NewFromConfig(cfg)

	// 1. Allocate a new Elastic IP
	allocOutput, err := client.AllocateAddress(ctx, &ec2.AllocateAddressInput{
		Domain: types.DomainTypeVpc,
	})
	if err != nil {
		log.Fatalf("failed to allocate EIP: %v", err)
	}
	publicIP := *allocOutput.PublicIp
	allocationID := *allocOutput.AllocationId
	fmt.Printf("Allocated EIP: %s\n", publicIP)

	// 2. Associate EIP with a target Instance
	targetInstanceID := "i-0abcd1234efgh5678"
	_, err = client.AssociateAddress(ctx, &ec2.AssociateAddressInput{
		AllocationId: &allocationID,
		InstanceId:   &targetInstanceID,
		// AllowReassociation is useful for moving an EIP that's already attached elsewhere
		AllowReassociation: aws.Bool(true),
	})
	if err != nil {
		log.Fatalf("failed to associate EIP: %v", err)
	}

	fmt.Printf("Successfully associated %s with instance %s\n", publicIP, targetInstanceID)
}
```

## Interview Questions

**Q: Why does AWS charge for an Elastic IP that is not associated with any instance?**
**A:** IPv4 addresses are a limited global resource. By charging for idle IPs, AWS incentivizes users to release addresses they aren't using, ensuring availability for others and promoting IPv6 adoption.

**Q: What happens to the auto-assigned public IP of an EC2 instance when you attach an Elastic IP?**
**A:** The auto-assigned public IP is released back into the Amazon public IP pool. Once released, you cannot get the same address back. The Elastic IP replaces it as the public-facing address.

**Q: Can you move an Elastic IP between different AWS Regions?**
**A:** No. Elastic IP addresses are Region-specific and cannot be moved or transferred to a different Region. You would need to allocate a new EIP in the target Region.

**Q: How do you implement a "Floating IP" pattern for high availability?**
**A:** You use a script or Lambda function to monitor health. Upon failure of the primary resource, the tool calls the `AssociateAddress` API to remap the EIP to a standby instance, ensuring minimal downtime for clients connecting via the IP.

**Q: What is the default limit of Elastic IPs per region?**
**A:** The default limit is **5 Elastic IPs per Region** per account. This can be increased by submitting a request through the Service Quotas console.
