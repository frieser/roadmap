---
tags: ['tools', 'roadmap']
---

## Summary
In Terraform, **Sensitive Outputs** are output values explicitly marked with `sensitive = true`. This configuration prevents the values from being displayed in plain text during `terraform plan` or `terraform apply` CLI execution, protecting secrets like passwords or API keys from appearing in CI/CD logs. However, **this does not encrypt the data in the state file**; sensitive values remain in plain text within `terraform.tfstate`, requiring strict state file access controls.

## Detailed Explanation

### The Problem: Accidental Exposure
By default, Terraform displays all defined `output` values at the end of an `apply` run. If an output contains a database password, private key, or API token, this secret becomes visible in:
1.  The terminal output of the operator running the command.
2.  CI/CD pipeline logs (e.g., GitHub Actions, Jenkins), which are often accessible to many developers.

### The Solution: `sensitive = true`
You can suppress this output by adding the `sensitive = true` argument to the output block. Terraform will redact the value in the CLI, showing `(sensitive value)` instead of the actual data.

### Critical Security Warning: State Files
The `sensitive` flag is **only a display configuration**. It does NOT encrypt or redact the value in the Terraform state file (`terraform.tfstate`).
*   **Fact**: Anyone with read access to the state file can read the sensitive values in plain text.
*   **Best Practice**: Always store state files remotely (e.g., AWS S3 with encryption at rest, Terraform Cloud) and strictly limit access permissions (IAM policies).

### Propagation in Modules
Terraform is strict about sensitive data handling to prevent accidental leaks:
1.  If a child module exports a sensitive value, the parent module **must** also mark the consuming output as sensitive.
2.  If you try to interpolate a sensitive value into a non-sensitive output, Terraform will return an error, forcing you to either mark the new output as sensitive or explicitly use the `nonsensitive()` function (use with caution).

### Code Examples

#### 1. Basic Sensitive Output (Terraform)
This is the standard way to protect a database password.

```hcl
resource "aws_db_instance" "default" {
  allocated_storage = 10
  engine            = "mysql"
  username          = "admin"
  password          = "supersecretpassword123" # In real usage, use a variable!
}

# Without 'sensitive = true', this would print the password in logs
output "db_password" {
  value       = aws_db_instance.default.password
  description = "The password for the database"
  sensitive   = true
}
```

**CLI Output:**
```text
Changes to Outputs:
  + db_password = (sensitive value)
```

#### 2. Handling Sensitive Outputs in Tests (Go/Terratest)
When writing automated tests for your infrastructure using Go and [Terratest](https://terratest.gruntwork.io/), you often need to verify that outputs are set correctly. Since `terraform output` masks these values, you need a way to read them in your tests.

Terratest's `terraform.Output` function automatically handles fetching outputs. For sensitive values, you simply use it as normal, but often relying on the underlying JSON inspection which bypasses the CLI masking for programmatic access.

```go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestSensitiveOutput(t *testing.T) {
	t.Parallel()

	terraformOptions := &terraform.Options{
		// The path to where your Terraform code is located
		TerraformDir: "../examples/sensitive-output",
		
		// Variables to pass to our Terraform code using -var options
		Vars: map[string]interface{}{
			"db_password": "test-password-123",
		},
	}

	// At the end of the test, run `terraform destroy` to clean up resources
	defer terraform.Destroy(t, terraformOptions)

	// Run `terraform init` and `terraform apply` and fail the test if there are any errors
	terraform.InitAndApply(t, terraformOptions)

	// Output fetches the output value. 
	// Note: Even though it is marked sensitive in HCL, Terratest can read it 
	// because it parses the machine-readable output (terraform output -json).
	actualPassword := terraform.Output(t, terraformOptions, "db_password")

	// Verify the sensitive output matches what we expect
	assert.Equal(t, "test-password-123", actualPassword)
}
```

#### 3. Debugging with `nonsensitive()` (Terraform)
If you absolutely need to expose a sensitive value (e.g., for debugging), you can use the `nonsensitive()` function. **Warning**: This defeats the protection mechanism.

```hcl
output "debug_password" {
  value     = nonsensitive(aws_db_instance.default.password)
  sensitive = false 
  # This will PRINT the password in the logs!
}
```

## Interview Questions

**Q: Does setting `sensitive = true` encrypt the data in the Terraform state file?**
**A:** No. It only masks the value in the CLI output (`terraform plan` / `apply`). The value is still stored in plain text in the `terraform.tfstate` file. This is why securing the state file (encryption at rest, strict access controls) is critical.

**Q: You are using a module that outputs a database password marked as sensitive. You want to output this password from your root module. What happens if you don't mark your root output as sensitive?**
**A:** Terraform will throw an error. It prevents you from accidentally exposing a sensitive value derived from another sensitive value. You must explicitly mark your root output as `sensitive = true` or use the `nonsensitive()` function to override the protection.

**Q: How can you view a sensitive output value without modifying the Terraform code?**
**A:** You can retrieve the value using the `terraform output` command with the `-json` flag, which outputs all values in JSON format, or by querying the specific output name (depending on Terraform version, this might still require `-raw` or `-json`). For example: `terraform output -json db_password`.
