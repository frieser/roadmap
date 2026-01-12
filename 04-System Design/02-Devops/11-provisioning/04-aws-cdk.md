---
---
# AWS CDK

The AWS Cloud Development Kit (AWS CDK) is an open-source software development framework to define cloud infrastructure in code and provision it through AWS CloudFormation.

## Core Concepts

*   **Constructs**: The basic building blocks of CDK applications. A construct represents a "cloud component" and encapsulates everything CloudFormation needs to create the component.
*   **Apps and Stacks**: An App is the root of the construct tree. Stacks represent a unit of deployment (mapping directly to CloudFormation stacks).
*   **Synthesis (Synth)**: The process of executing your CDK code and generating a CloudFormation template.
*   **Bootstrap**: The process of preparing an AWS environment for deployment (creating resources like an S3 bucket for assets).

## Go Example: Defining a Stack

```go
package main

import (
	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awss3"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

func NewMyStack(scope constructs.Construct, id string, props *awscdk.StackProps) awscdk.Stack {
	stack := awscdk.NewStack(scope, &id, props)
	awss3.NewBucket(stack, jsii.String("MyBucket"), &awss3.BucketProps{
		Versioned: jsii.Bool(true),
	})
	return stack
}

func main() {
	app := awscdk.NewApp(nil)
	NewMyStack(app, "MyStack", nil)
	app.Synth(nil)
}
```
