#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ecs']
---

## Summary
An **Amazon ECS Service** allows you to run and maintain a specified number of instances of a **Task Definition** simultaneously in an ECS cluster. It acts as the scheduler and manager for long-running applications (stateless or stateful), ensuring that the desired number of tasks are always healthy and running. It integrates seamlessly with Elastic Load Balancing (ELB) for traffic distribution and Service Discovery for internal communication.

## Detailed Explanation

### 1. Core Concept
While a **Task** is a running instance of a container (or set of containers), a **Service** is the configuration that ensures those tasks stay running. If a task fails or stops for any reason, the ECS Service scheduler automatically launches a new instance to replace it, maintaining the **Desired Count**.

### 2. Key Features

#### Load Balancing Integration
ECS Services can be associated with an Application Load Balancer (ALB) or Network Load Balancer (NLB). The service automatically registers new tasks with the Load Balancer's Target Group and deregisters them when they stop.
- **Dynamic Port Mapping**: When using the EC2 launch type with `bridge` networking, you can set the host port to 0. The ALB automatically detects the ephemeral port assigned to the container.

#### Deployment Strategies
ECS Services manage how updates to the application (new Task Definition revisions) are rolled out:
- **Rolling Update (ECS Default)**: Gradually replaces old tasks with new ones. Controlled by:
    - `minHealthyPercent`: The lower limit on the number of running tasks during a deployment (e.g., 50% means at least half of the desired count must be running).
    - `maxPercent`: The upper limit on the number of running tasks (e.g., 200% means you can double the task count during deployment).
- **Blue/Green (AWS CodeDeploy)**: Shifts traffic from the old version (Blue) to the new version (Green) after verification. It is safer but requires more setup.

#### Service Auto Scaling
Distinct from Cluster Auto Scaling (which scales EC2 instances), **Service Auto Scaling** increases or decreases the **Desired Count** of tasks.
- **Target Tracking**: Scales based on a specific metric value (e.g., "Keep average CPU utilization at 70%").
- **Step Scaling**: Scales based on CloudWatch Alarms breaching thresholds.

#### Service Discovery
For service-to-service communication within a VPC, ECS integrates with **AWS Cloud Map**. It creates a private DNS namespace (e.g., `my-app.local`) and automatically updates DNS records as tasks launch and stop, allowing services to find each other by name.

### 3. Launch Types
- **Fargate**: Serverless. You specify CPU/Memory, and AWS manages the underlying infrastructure. The Service manages Fargate tasks.
- **EC2**: You manage the EC2 instances (Cluster). The Service places tasks on your instances based on placement strategies (e.g., binpack, spread).

### Go Example: Creating an ECS Service
The following example demonstrates how to create an ECS Service using the AWS SDK for Go v2.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/ecs"
	"github.com/aws/aws-sdk-go-v2/service/ecs/types"
)

func main() {
	// Load AWS configuration
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := ecs.NewFromConfig(cfg)

	// Define the CreateService input
	input := &ecs.CreateServiceInput{
		ServiceName:    aws.String("my-go-web-service"),
		Cluster:        aws.String("my-cluster"),
		TaskDefinition: aws.String("my-go-app-task:1"), // Family:Revision
		DesiredCount:   aws.Int32(2),
		LaunchType:     types.LaunchTypeFargate,
		
		// Network Configuration (Required for Fargate)
		NetworkConfiguration: &types.NetworkConfiguration{
			AwsvpcConfiguration: &types.AwsVpcConfiguration{
				Subnets:        []string{"subnet-12345678", "subnet-87654321"},
				SecurityGroups: []string{"sg-0a1b2c3d4e5f"},
				AssignPublicIp: types.AssignPublicIpEnabled,
			},
		},

		// Load Balancer Configuration
		LoadBalancers: []types.LoadBalancer{
			{
				TargetGroupArn: aws.String("arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-tg/6d0ecf831eec9f09"),
				ContainerName:  aws.String("web-server"),
				ContainerPort:  aws.Int32(8080),
			},
		},
	}

	// Create the Service
	output, err := client.CreateService(context.TODO(), input)
	if err != nil {
		log.Fatalf("failed to create service, %v", err)
	}

	fmt.Printf("Created Service: %s (ARN: %s)\n", *output.Service.ServiceName, *output.Service.ServiceArn)
}
```

## Interview Questions

**Q: What is the difference between ECS Service Auto Scaling and Cluster Auto Scaling?**
**A:** Service Auto Scaling adjusts the number of **tasks** (containers) running in a service based on load (CPU/Memory). Cluster Auto Scaling (Capacity Providers) adjusts the number of **EC2 instances** in the underlying cluster to ensure there is enough infrastructure to run those tasks.

**Q: Explain `minHealthyPercent` and `maxPercent` in a Rolling Update.**
**A:** `minHealthyPercent` ensures availability during deployment (e.g., 50% means half the tasks must remain running). `maxPercent` limits resource usage (e.g., 200% means you can temporarily double the tasks to spin up new ones before killing old ones).

**Q: How does an ECS Service handle a failed task?**
**A:** The ECS Service scheduler constantly monitors the state of tasks. If a task stops (exits) or fails an ELB health check, the scheduler notices the running count is below the desired count and initiates a new task launch to replace it.

**Q: Can you run a stateful application as an ECS Service?**
**A:** Yes, using EFS (Elastic File System) volumes. You can mount an EFS file system to your tasks, allowing them to share persistent data even if tasks are killed and replaced.
