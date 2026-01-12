---
---
# Terraform

Terraform is an open-source infrastructure as code software tool that provides a consistent CLI workflow to manage hundreds of cloud services.

## Core Concepts

*   **HCL (HashiCorp Configuration Language)**: A human-readable language for describing infrastructure.
*   **State File**: Terraform keeps track of your infrastructure in a state file (`.tfstate`). This file is used to map real-world resources to your configuration, keep track of metadata, and improve performance for large infrastructures.
*   **Providers**: Plugins that allow Terraform to communicate with cloud providers (AWS, Azure, GCP), SaaS providers, and other APIs.
*   **Modules**: Reusable containers for multiple resources that are used together. Modules allow you to group resources and reuse them across different environments.
*   **Plan and Apply**: The core workflow consists of `terraform plan` (to see what will happen) and `terraform apply` (to make it happen).

## Go Example: Testing with Terratest

Go developers often use `terratest` to verify their Terraform code.

```go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformS3Example(t *testing.T) {
	opts := &terraform.Options{
		TerraformDir: "../examples/s3-bucket",
		Vars: map[string]interface{}{
			"bucket_name": "my-terratest-bucket",
		},
	}

	defer terraform.Destroy(t, opts)
	terraform.InitAndApply(t, opts)

	bucketID := terraform.Output(t, opts, "bucket_id")
	assert.NotEmpty(t, bucketID)
}
```
