---
tags: ['tools', 'roadmap']
---

# Terraform Variable Validation Rules

## Summary
Terraform **Validation Rules** allow you to define custom constraints for input variables to ensure they meet specific criteria before infrastructure is provisioned. By using a `validation` block within a `variable` definition, you can enforce formats (like AMI IDs or IP addresses), value ranges, or allowed lists. This "fail-fast" mechanism catches configuration errors during the `plan` phase, preventing costly or dangerous misconfigurations from reaching the `apply` stage.

## Detailed Explanation

Validation rules act as a gatekeeper for your Terraform modules. While `type` constraints (e.g., `string`, `list`, `map`) ensure the data structure is correct, `validation` blocks ensure the *content* of that data is valid.

### Syntax
A `validation` block requires two arguments:
1.  **`condition`**: An expression that returns `true` if the value is valid, or `false` if it is invalid. This expression can **only** refer to the variable itself (e.g., `var.my_variable`).
2.  **`error_message`**: A string to display to the user if the condition is false. It must start with a capital letter and generally be a full sentence.

```hcl
variable "image_id" {
  type        = string
  description = "The id of the machine image (AMI) to use for the server."

  validation {
    condition     = length(var.image_id) > 4 && substr(var.image_id, 0, 4) == "ami-"
    error_message = "The image_id value must be a valid AMI id, starting with \"ami-\"."
  }
}
```

### Common Validation Functions
You often use Terraform built-in functions within the `condition` expression:
*   **`length()`**: Check the number of characters or items.
*   **`regex()`** or **`can(regex(...))`**: Validate against a regular expression pattern.
*   **`contains()`**: Check if a value exists in a defined list (allow-listing).
*   **`substr()`**: Check prefixes or specific substrings.
*   **`can()`**: Safely evaluate an expression that might otherwise fail (e.g., checking if a string can be parsed as a CIDR).

### Examples

#### 1. Allow-List (Enum)
Ensure a variable is one of a specific set of values (e.g., environment names).

```hcl
variable "environment" {
  type        = string
  description = "Deployment environment (dev, stage, prod)"

  validation {
    condition     = contains(["dev", "stage", "prod"], var.environment)
    error_message = "Environment must be one of: dev, stage, prod."
  }
}
```

#### 2. Regex Pattern (IP Address)
Validate that a string follows a specific format, such as an IP address.

```hcl
variable "server_ip" {
  type        = string
  description = "Server IP address"

  validation {
    condition     = can(regex("^(?:[0-9]{1,3}\\.){3}[0-9]{1,3}$", var.server_ip))
    error_message = "Must be a valid IPv4 address."
  }
}
```

#### 3. Numeric Range
Ensure a port number or count falls within a safe range.

```hcl
variable "listener_port" {
  type        = number
  description = "Load balancer listener port"

  validation {
    condition     = var.listener_port >= 80 && var.listener_port <= 65535
    error_message = "Port number must be between 80 and 65535."
  }
}
```

## Go Application (Testing with Terratest)

While validation rules are written in HCL, **Terratest** (a Go library) is the industry standard for verifying them. You write a test that intentionally passes an invalid value and asserts that Terraform fails with the expected error message.

This ensures your validation logic actually protects your module as intended.

### Code Example
The following Go code demonstrates how to test the `environment` variable validation from the example above.

```go
package test

import (
	"testing"

	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformValidationRules(t *testing.T) {
	// 1. Define Terraform options
	terraformOptions := &terraform.Options{
		// Path to the Terraform code
		TerraformDir: "../examples/validation-demo",

		// 2. Pass an INVALID value to trigger the validation rule
		Vars: map[string]interface{}{
			"environment": "invalid-env", // Should only allow dev, stage, prod
		},
	}

	// 3. Clean up resources (though InitAndPlan shouldn't create any)
	defer terraform.Destroy(t, terraformOptions)

	// 4. Run `terraform init` and `terraform plan`.
	// Since we expect it to fail, we use InitAndPlanE (E for Error) which returns the error.
	_, err := terraform.InitAndPlanE(t, terraformOptions)

	// 5. Assert that an error occurred
	assert.Error(t, err)

	// 6. Assert that the error message contains our custom validation message
	// This confirms the failure was due to OUR rule, not something else.
	assert.Contains(t, err.Error(), "Environment must be one of: dev, stage, prod")
}
```

**Why this is useful in Go:**
*   **Regression Testing**: Ensures no one accidentally removes or breaks a critical safety check in the HCL.
*   **Documentation**: The test serves as executable documentation of what is considered "invalid" input.

## Interview Questions

### Q: Can a validation condition reference other variables?
**A:** No, within a `variable` block, the `validation` `condition` can only reference the variable itself (e.g., `var.my_var`). It cannot depend on `var.other_var` or resources. To enforce rules depending on multiple values, you must use `precondition` or `postcondition` blocks within `lifecycle` blocks of resources or outputs.

### Q: When does Terraform check validation rules?
**A:** Validation rules are checked during the `terraform plan` phase (and `apply` if plan is skipped). If the validation fails, Terraform aborts execution immediately, preventing any changes to the state or infrastructure.

### Q: What is the difference between `type` constraints and `validation` rules?
**A:** `type` restricts the **data structure** (e.g., ensuring a value is a list of strings), while `validation` restricts the **logical value** (e.g., ensuring the strings in that list are valid email addresses). Validation allows for much more granular and business-logic-specific control than types alone.

### Q: Can you have multiple validation blocks for a single variable?
**A:** Yes, you can define multiple `validation` blocks for a single variable. Terraform will check all of them, and if *any* condition fails, it will display the corresponding error message. This allows you to check for distinct requirements (e.g., one rule for length, another for character set) and give specific feedback.
