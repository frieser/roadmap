---
---
# Pulumi

Pulumi is an infrastructure as code tool that allows you to use familiar programming languages to define and deploy cloud infrastructure.

## Core Concepts

*   **Real Programming Languages**: Use Go, Python, TypeScript, or C# instead of DSLs like HCL. This enables the use of standard IDEs, testing frameworks, and abstraction patterns.
*   **State-Based**: Like Terraform, Pulumi maintains a state of the infrastructure it manages. By default, this state is stored in the Pulumi Cloud, but it can also be self-managed (e.g., in S3).
*   **Stacks**: A stack is an isolated, independently configurable instance of a Pulumi program. You might have stacks for `dev`, `staging`, and `prod`.
*   **Automation API**: A programmatic interface for running Pulumi programs, allowing you to embed infrastructure automation inside your application logic.

## Go Example: Provisioning S3

```go
package main

import (
	"github.com/pulumi/pulumi-aws/sdk/v6/go/aws/s3"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {
		bucket, err := s3.NewBucket(ctx, "my-bucket", &s3.BucketArgs{
			Acl: pulumi.String("private"),
		})
		if err != nil {
			return err
		}
		ctx.Export("bucketName", bucket.ID())
		return nil
	})
}
```
