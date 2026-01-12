#AWS
#Cloud

---
tags: ['aws', 'roadmap']
---

## Summary
In Amazon ECS, a **Task Definition** is a blueprint (JSON file) that describes how one or more containers should be launched, including their images, resources, and configurations. A **Task** is the actual running instantiation of that definition within an ECS cluster. This decoupling allows for versioning and consistent deployments across different environments.

## Detailed Explanation

### Task Definitions
A Task Definition is required to run Docker containers in Amazon ECS. It defines the parameters for your application, such as:
- **Family**: The name of the task definition (versioned automatically).
- **Container Definitions**: An array of container objects (up to 10) describing image, CPU, memory, port mappings, and environment variables.
- **Network Mode**: Defines how containers are networked (e.g., `bridge`, `host`, `awsvpc`). `awsvpc` is the default and recommended for Fargate.
- **Task Size**: The total CPU and memory available to the task.

### Task Role vs. Execution Role
- **Task Role (`taskRoleArn`)**: Provides permissions to the containers themselves. Use this for your application code to interact with AWS services like S3 or DynamoDB.
- **Task Execution Role (`executionRoleArn`)**: Provides permissions to the ECS agent and Fargate infrastructure. It allows them to pull images from ECR, send logs to CloudWatch, and access secrets from Secrets Manager.

### Tasks
A **Task** is the unit of work in ECS. You can run a standalone task (one-off job) or use an ECS Service to maintain a desired number of tasks (long-running application).

### Go Example: Registering a Task Definition
The following example demonstrates how to register a new task definition using the AWS SDK for Go v2.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/ecs"
	"github.com/aws/aws-sdk-go-v2/service/ecs/types"
	"github.com/aws/aws-sdk-go-v2/aws"
)

func main() {
	// Load AWS configuration
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := ecs.NewFromConfig(cfg)

	// Define the task definition
	input := &ecs.RegisterTaskDefinitionInput{
		Family: aws.String("my-go-app-task"),
		ContainerDefinitions: []types.ContainerDefinition{
			{
				Name:  aws.String("web-server"),
				Image: aws.String("my-repo/my-go-app:latest"),
				PortMappings: []types.PortMapping{
					{
						ContainerPort: aws.Int32(8080),
						HostPort:      aws.Int32(8080),
						Protocol:      types.TransportProtocolTcp,
					},
				},
				Cpu:    aws.Int32(256),
				Memory: aws.Int32(512),
			},
		},
		RequiresCompatibilities: []types.Compatibility{
			types.CompatibilityFargate,
		},
		NetworkMode: types.NetworkModeAwsvpc,
		Cpu:         aws.String("256"),
		Memory:      aws.String("512"),
		ExecutionRoleArn: aws.String("arn:aws:iam::123456789012:role/ecsTaskExecutionRole"),
		TaskRoleArn:      aws.String("arn:aws:iam::123456789012:role/myAppTaskRole"),
	}

	// Register the task definition
	output, err := client.RegisterTaskDefinition(context.TODO(), input)
	if err != nil {
		log.Fatalf("failed to register task definition, %v", err)
	}

	fmt.Printf("Registered Task Definition: %s (Revision %d)\n", 
		*output.TaskDefinition.Family, output.TaskDefinition.Revision)
}
```

## Interview Questions

**Q: What is the main difference between a Task Definition and a Task?**
**A:** A Task Definition is a blueprint or template (JSON) that defines how containers should run, while a Task is the actual running instance of that definition in the cluster.

**Q: Why would you use multiple containers in a single Task Definition?**
**A:** You use multiple containers when they share a common lifecycle and need to communicate over localhost or share storage (sidecar pattern). Common examples include log forwarders, proxies (like Envoy), or service meshes.

**Q: What is the purpose of the Task Execution Role?**
**A:** It allows the ECS agent and Fargate to perform actions on your behalf, such as pulling container images from ECR and sending container logs to CloudWatch Logs.

**Q: What happens if you update a Task Definition that is currently being used by an ECS Service?**
**A:** Task Definitions are immutable and versioned by "revisions". Updating a Task Definition creates a new revision. To apply the change, you must update the ECS Service to use the new revision, which triggers a rolling update of the tasks.

**Q: Which network mode is required for AWS Fargate?**
**A:** The `awsvpc` network mode is required for tasks using the Fargate launch type. It gives each task its own elastic network interface (ENI) and a private IP address.
