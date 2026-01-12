#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'auto-scaling']
---

## Summary
AWS **Launch Templates** are the modern and recommended way to define instance configurations for EC2 Auto Scaling Groups (ASG). Succeeding the legacy Launch Configurations, they provide a flexible, versioned template that specifies the AMI, instance type, security groups, and other launch parameters. Their primary advantages include support for versioning, mixed instance policies (combining Spot and On-Demand), and access to the latest EC2 features.

## Detailed Explanation

### Evolution from Launch Configurations
Launch Configurations were the original method for defining how an ASG should launch instances. However, they had significant drawbacks:
- **Immutability**: Once created, a Launch Configuration cannot be modified. Any change requires creating a new one and updating the ASG.
- **Limited Features**: They do not support modern EC2 capabilities like multiple instance types, T2/T3 Unlimited, or EBS volume tagging at launch.
- **Deprecation**: AWS has ceased support for new instance types in Launch Configurations as of January 2023 and restricted their creation for new accounts.

### Key Features of Launch Templates
1. **Versioning**: Launch Templates allow you to maintain multiple versions of a configuration. You can designate a specific version as "Default" or configure an ASG to always use the "Latest" version, facilitating seamless deployments and rollbacks.
2. **Mixed Instances Policy**: This allows an ASG to launch a combination of On-Demand and Spot instances across multiple instance types (e.g., `t3.micro` and `t3.small`). This optimizes cost and improves availability by diversifying instance pools.
3. **Parameter Subsets**: You can define a base template and override only specific parameters when creating new versions or launching instances.
4. **Modern EC2 Integration**:
    * **Systems Manager (SSM) Parameters**: Dynamically reference the latest AMI ID.
    * **Advanced Networking**: Support for multiple network interfaces and Elastic Fabric Adapter (EFA).
    * **Tagging**: Automatic tagging of instances and EBS volumes during the launch process.

### Go Application Example
Using the AWS SDK for Go v2 to programmatically create a Launch Template:

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

func createTemplate(ctx context.Context, client *ec2.Client) {
	input := &ec2.CreateLaunchTemplateInput{
		LaunchTemplateName: aws.String("prod-web-server-v1"),
		LaunchTemplateData: &types.RequestLaunchTemplateData{
			ImageId:      aws.String("ami-0c55b159cbfafe1f0"), // Amazon Linux 2
			InstanceType: types.InstanceTypeT3Micro,
			KeyName:      aws.String("deploy-key"),
			SecurityGroupIds: []string{
				"sg-0abc123456789def0",
			},
			TagSpecifications: []types.LaunchTemplateTagSpecificationRequest{
				{
					ResourceType: types.ResourceTypeInstance,
					Tags: []types.Tag{
						{Key: aws.String("Project"), Value: aws.String("Roadmap")},
					},
				},
			},
		},
	}

	result, err := client.CreateLaunchTemplate(ctx, input)
	if err != nil {
		log.Fatalf("failed to create launch template: %v", err)
	}

	fmt.Printf("Created Template: %s\n", *result.LaunchTemplate.LaunchTemplateId)
}

func main() {
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config: %v", err)
	}

	client := ec2.NewFromConfig(cfg)
	createTemplate(context.TODO(), client)
}
```

## Interview Questions

**Q: Why should you prefer Launch Templates over Launch Configurations?**
**A:** Launch Templates are the modern standard and support **versioning**, which simplifies updates and rollbacks. Crucially, they enable the **Mixed Instances Policy** (using multiple instance types and Spot/On-Demand mixes in one ASG), whereas Launch Configurations are immutable and lack support for modern features like EBS tagging or T2/T3 Unlimited credits.

**Q: Can you update an existing Launch Template?**
**A:** You cannot modify an existing version of a template because versions themselves are immutable for auditability. Instead, you create a **new version** of the template with the updated parameters. You can then set this new version as the "Default" for your Auto Scaling Group.

**Q: How do Launch Templates facilitate cost optimization?**
**A:** By using a **Mixed Instances Policy**, you can define an ASG that launches a base amount of On-Demand capacity and scales further using **Spot Instances**. You can also specify multiple instance types to increase the chances of fulfilling Spot requests, significantly reducing overall infrastructure costs.

**Q: What is the benefit of using SSM Parameters with Launch Templates?**
**A:** Instead of hardcoding an AMI ID (which changes frequently), you can reference an **SSM Parameter** (e.g., `resolve:ssm:/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2`). This ensures the Launch Template always uses the latest patched version of the OS without requiring manual template updates.
