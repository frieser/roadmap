#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'vpc']
---

## Summary
A **Public Subnet** is a subnet in an AWS Virtual Private Cloud (VPC) that is configured to allow direct access to and from the public internet. This is achieved by associating the subnet with a route table that contains a default route (`0.0.0.0/0`) pointing to an **Internet Gateway (IGW)**. Resources in a public subnet typically require a public IPv4 address or an Elastic IP to communicate with external networks.

## Detailed Explanation

In AWS, subnets are not "public" or "private" by their inherent properties at creation; rather, their classification depends on their **routing configuration**.

### 1. The Internet Gateway (IGW)
An Internet Gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet.
- **Attachment**: An IGW must be created and then explicitly attached to a VPC.
- **Function**: It performs Network Address Translation (NAT) for instances that have been assigned public IPv4 addresses.

### 2. Routing to the Internet
For a subnet to be considered public, its associated **Route Table** must have a specific entry:
- **Destination**: `0.0.0.0/0` (for IPv4) or `::/0` (for IPv6).
- **Target**: The ID of the Internet Gateway (e.g., `igw-xxxxxxxx`).

Any traffic originating from the subnet that is not destined for the local VPC range will be forwarded to the Internet Gateway.

### 3. Public IP Addresses
Even with a route to the IGW, an instance cannot communicate with the internet unless it has a public IP address.
- **Auto-assign Public IP**: A subnet attribute that can be enabled so that any instance launched into it automatically receives a public IPv4 address from Amazon's pool.
- **Elastic IP (EIP)**: A static, reserved public IPv4 address that can be manually associated with an instance.

### 4. Common Use Cases
- **Web Servers**: Hosting public-facing websites or APIs.
- **Load Balancers**: Internet-facing Application Load Balancers (ALBs) must be placed in public subnets.
- **NAT Gateways**: While NAT Gateways provide internet access to private subnets, the NAT Gateway itself must reside in a public subnet to reach the IGW.

### Go Context
When automating AWS infrastructure with the **AWS SDK for Go v2**, you typically perform these steps to set up a public subnet:

```go
package main

import (
	"context"
	"fmt"
	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/ec2"
)

func main() {
	ctx := context.TODO()
	cfg, _ := config.LoadDefaultConfig(ctx)
	client := ec2.NewFromConfig(cfg)

	subnetID := "subnet-12345"
	igwID := "igw-67890"
	routeTableID := "rtb-54321"

	// 1. Create a route to the Internet Gateway
	_, err := client.CreateRoute(ctx, &ec2.CreateRouteInput{
		RouteTableId:         aws.String(routeTableID),
		DestinationCidrBlock: aws.String("0.0.0.0/0"),
		GatewayId:            aws.String(igwID),
	})
	if err != nil {
		fmt.Printf("Error creating route: %v\n", err)
	}

	// 2. Enable auto-assign public IP on launch for the subnet
	_, err = client.ModifySubnetAttribute(ctx, &ec2.ModifySubnetAttribute{
		SubnetId: aws.String(subnetID),
		MapPublicIpOnLaunch: &ec2.AttributeBooleanValue{
			Value: aws.Bool(true),
		},
	})
	if err != nil {
		fmt.Printf("Error modifying subnet attribute: %v\n", err)
	}
}
```

## Interview Questions

**Q: What is the defining characteristic of a public subnet in AWS?**
**A:** A public subnet is defined by having a route in its associated route table that points default traffic (`0.0.0.0/0`) to an Internet Gateway (IGW).

**Q: Can an instance in a public subnet communicate with the internet if it doesn't have a public IP address?**
**A:** No. While the routing allows the traffic to leave the subnet towards the IGW, the IGW cannot route traffic back to the instance without a public IP (as it doesn't know how to map the private IP back from the internet).

**Q: Does a public subnet require a NAT Gateway?**
**A:** No. Public subnets use an Internet Gateway for direct access. NAT Gateways are used by instances in **private subnets** to access the internet while remaining unreachable from the internet.

**Q: What is the difference between an Internet Gateway and a NAT Gateway?**
**A:** An IGW allows both inbound and outbound communication (two-way) for instances with public IPs. A NAT Gateway allows only outbound communication (one-way) for instances in private subnets, masking their private IPs behind the NAT Gateway's public IP.
