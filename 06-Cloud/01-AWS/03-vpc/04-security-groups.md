#AWS
#Cloud

---
tags: ['aws', 'roadmap', 'vpc', 'security']
---

## Summary
Security Groups act as a virtual firewall for your EC2 instances to control inbound and outbound traffic. Operating at the **instance level** (ENI), they are **stateful**, meaning that if you send a request from your instance, the response traffic for that request is allowed to reach the instance regardless of inbound rules. Security groups contain only "allow" rules; there is no way to explicitly "deny" traffic.

## Detailed Explanation

### Core Characteristics
*   **Instance Level Security**: Security groups are associated with Network Interfaces (ENI) rather than subnets. This allows for granular control per instance.
*   **Stateful Nature**: This is a critical feature. If an inbound packet is allowed, the outbound response is automatically allowed (and vice versa), even if no explicit rule exists for the return path.
*   **Default Behavior**: 
    *   New security groups start with **no inbound rules** (implicit deny all).
    *   They start with a **default outbound rule** that allows all traffic.
*   **Rules**: You can specify protocol (TCP, UDP, ICMP), port range, and source/destination (CIDR block, another security group, or a prefix list).

### Security Group vs. Network ACL (NACL)
| Feature | Security Group | Network ACL |
| :--- | :--- | :--- |
| **Level** | Instance/ENI | Subnet |
| **State** | Stateful | Stateless |
| **Rules** | Allow rules only | Allow and Deny rules |
| **Evaluation** | All rules evaluated | Rules evaluated in order (numbered) |

### Referencing Security Groups
A powerful feature is the ability to reference other security groups as a source or destination. This creates a logical connection rather than relying on IP addresses. For example, a Web Server group can allow traffic from a Load Balancer group without needing to know the LB's private IPs.

### Go Implementation (AWS SDK v2)
In Go, you typically manage security groups using the `github.com/aws/aws-sdk-go-v2/service/ec2` package.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/ec2"
	"github.com/aws/aws-sdk-go-v2/service/ec2/types"
)

func main() {
	ctx := context.TODO()
	cfg, err := config.LoadDefaultConfig(ctx, config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := ec2.NewFromConfig(cfg)

	// 1. Create the Security Group
	createOutput, err := client.CreateSecurityGroup(ctx, &ec2.CreateSecurityGroupInput{
		Description: aws.String("Allow SSH and Web traffic"),
		GroupName:   aws.String("web-server-sg"),
		VpcId:       aws.String("vpc-12345678"),
	})
	if err != nil {
		log.Fatalf("failed to create security group, %v", err)
	}

	groupID := createOutput.GroupId
	fmt.Printf("Created Security Group: %s\n", *groupID)

	// 2. Authorize Ingress Rules (SSH and HTTP)
	_, err = client.AuthorizeSecurityGroupIngress(ctx, &ec2.AuthorizeSecurityGroupIngressInput{
		GroupId: groupID,
		IpPermissions: []types.IpPermission{
			{
				IpProtocol: aws.String("tcp"),
				FromPort:   aws.Int32(22),
				ToPort:     aws.Int32(22),
				IpRanges: []types.IpRange{
					{CidrIp: aws.String("203.0.113.0/24"), Description: aws.String("Office IP")},
				},
			},
			{
				IpProtocol: aws.String("tcp"),
				FromPort:   aws.Int32(80),
				ToPort:     aws.Int32(80),
				IpRanges: []types.IpRange{
					{CidrIp: aws.String("0.0.0.0/0")},
				},
			},
		},
	})
	if err != nil {
		log.Fatalf("failed to authorize ingress, %v", err)
	}
}
```

## Interview Questions

**Q: What does it mean that Security Groups are "stateful"?**
**A:** It means that if an initial request is allowed through the firewall, the response traffic is automatically permitted regardless of any rules in the opposite direction. If you allow inbound traffic on port 80, the instance can send the response back to the client even if there is no outbound rule for it.

**Q: Can you explicitly deny an IP address in a Security Group?**
**A:** No. Security Groups only support "allow" rules. If an IP or traffic pattern is not explicitly allowed, it is denied by default. To explicitly deny a specific IP, you must use a Network ACL (NACL).

**Q: How do you allow two instances in different subnets to communicate if they are in the same Security Group?**
**A:** By adding a rule to the Security Group that allows the required protocol/port, and setting the **Source** of the rule to the Security Group's own ID (`sg-xxxxxx`). This is called "referencing itself" or "security group nesting".

**Q: What is the main difference between a Security Group and a Network ACL?**
**A:** Security Groups are stateful and applied at the instance (ENI) level, while NACLs are stateless and applied at the subnet level. Security Groups only support allow rules, whereas NACLs support both allow and deny rules and evaluate them in numbered order.
