---
---
# CloudFormation

AWS CloudFormation is a service that helps you model and set up your Amazon Web Services resources.

## Core Concepts

*   **Templates**: JSON or YAML files that describe the resources you want to provision in AWS.
*   **Stacks**: A collection of AWS resources that you can manage as a single unit. When you create, update, or delete a stack, CloudFormation manages the resources accordingly.
*   **Change Sets**: Before making changes to a stack, you can create a change set to see how your changes might impact your running resources.
*   **Drift Detection**: Allows you to detect whether a stack's actual configuration has been changed outside of CloudFormation.
*   **Nested Stacks**: Stacks created as part of other stacks, allowing for modularity and reuse.

## Go Example: Managing Stacks via SDK

```go
package main

import (
	"context"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/cloudformation"
)

func main() {
	ctx := context.TODO()
	cfg, _ := config.LoadDefaultConfig(ctx)
	client := cloudformation.NewFromConfig(cfg)

	_, err := client.CreateStack(ctx, &cloudformation.CreateStackInput{
		StackName:    pulumi.String("MyStack"),
		TemplateBody: pulumi.String(`{"Resources": {"MyS3Bucket": {"Type": "AWS::S3::Bucket"}}}`),
	})
	if err != nil {
		panic(err)
	}
}
```
