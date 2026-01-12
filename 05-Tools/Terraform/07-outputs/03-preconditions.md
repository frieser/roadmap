---
tags: ['tools', 'roadmap']
---

## Summary
Terraform output preconditions are validation mechanisms introduced in Terraform 1.2 that allow module authors to define guarantees for the values they return. By adding a `precondition` block to an `output`, you can ensure that the data being passed to downstream consumers meets specific criteria (e.g., a server is actually healthy, a URL is valid). If the condition fails, Terraform raises an error and halts the operation, preventing the propagation of invalid state to other parts of the infrastructure.

## Detailed Explanation

### Concept and Purpose
In a modular Infrastructure as Code (IaC) architecture, outputs are the interface through which a module exposes data. Before preconditions were available, if a resource was created successfully but with unexpected properties (e.g., an empty string where an IP address was expected), that invalid data would silently propagate to other modules, causing failures much later in the dependency chain.

**Preconditions** act as a contract:
1.  **Validation**: They verify the integrity of the data *after* resources are created but *before* the output is returned.
2.  **Safety**: They protect consuming modules from receiving bad data.
3.  **Debugging**: They allow authors to provide custom, context-aware error messages.

### Syntax
A `precondition` block is nested directly within an `output` block. It requires:
*   `condition`: An expression that evaluates to `true` or `false`.
*   `error_message`: A string displayed to the user if the condition is `false`.

```hcl
output "api_endpoint" {
  value = "https://${aws_lb.main.dns_name}/v1"
  description = "The verified API endpoint"

  precondition {
    condition     = aws_lb.main.zone_id != ""
    error_message = " The Load Balancer has not been assigned a Zone ID yet."
  }
}
```

### Preconditions vs. Postconditions vs. Variable Validation

| Feature | Scope | Execution Time | Purpose |
| :--- | :--- | :--- | :--- |
| **Variable Validation** | `variable` | Before Apply (early) | Validates **inputs** from the user. |
| **Lifecycle Precondition** | `resource`, `data` | Before Resource Creation | Verifies assumptions about the environment or dependencies. |
| **Lifecycle Postcondition** | `resource`, `data` | After Resource Creation | Verifies that the created resource works as expected. |
| **Output Precondition** | `output` | After Resource Creation | Verifies the **result** being returned to the consumer. |

### HCL Example: Ensuring Valid Infrastructure
Imagine a module that creates a web server. You want to output its IP address, but only if the server is actually in a "running" state.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
}

output "server_ip" {
  value = aws_instance.web.public_ip

  precondition {
    condition     = aws_instance.web.public_ip != "" && aws_instance.web.instance_state == "running"
    error_message = "Server is not running or does not have a public IP."
  }
}
```

## Go Application: Testing Preconditions with Terratest
While the code above is HCL, the robust way to verify these preconditions works is by using **Go** and the **Terratest** library. You can write a test that intentionally triggers a failure condition and asserts that Terraform catches it.

### Scenario
We want to test that our Terraform code correctly fails when an output precondition is not met.

```go
package test

import (
	"testing"

	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformOutputPrecondition(t *testing.T) {
	t.Parallel()

	terraformOptions := &terraform.Options{
		// The path to where our Terraform code is located
		TerraformDir: "../examples/terraform-precondition-failure",
		
		// Variables to force a failure scenario (if your TF code supports it)
		// Or assume the TF code is designed to fail for this test
		Vars: map[string]interface{}{
			"force_invalid_state": true,
		},
	}

	// Defer the destroy to clean up resources
	defer terraform.Destroy(t, terraformOptions)

	// Run 'terraform init' and 'terraform apply'.
	// We expect this to FAIL because of the precondition.
	_, err := terraform.InitAndApplyE(t, terraformOptions)

	// Assert that we got an error
	assert.Error(t, err)

	// Assert that the error message contains our custom precondition message
	assert.Contains(t, err.Error(), "Server is not running or does not have a public IP")
}
```

### Explanation
1.  **`InitAndApplyE`**: We use the "E" version of the function (returning an error) instead of failing the test immediately.
2.  **`assert.Error`**: We verify that Terraform indeed returned an error (exit code non-zero).
3.  **`assert.Contains`**: We specifically check that the error log contains the text defined in the `error_message` of the `precondition` block. This confirms that it was our logic that stopped the deployment, not a random crash.

## Interview Questions

### Q: What is the main difference between a `variable` validation block and an `output` precondition?
**A:** Scope and Timing. `variable` validation checks the **inputs** provided by the user *before* Terraform creates any resources (during the planning phase). `output` preconditions check the **results** of the infrastructure creation *after* resources have been provisioned but before the module returns values to the user.

### Q: If an output precondition fails, what happens to the resources created during that `apply`?
**A:** The resources are **not** rolled back automatically. The `terraform apply` fails with an error, and the state file records the resources that were successfully created. However, the output value itself is not added to the state (or passed to parent modules), effectively preventing the rest of the configuration from using that invalid data.

### Q: Can I use `precondition` blocks to validate data from `data` sources?
**A:** Yes. You can place a `precondition` block inside a `lifecycle` block within a `data` source to verify assumptions about existing infrastructure (e.g., checking that a specific AMI exists) or inside an `output` block to validate the data fetched by that data source before exposing it.
