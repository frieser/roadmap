---
tags: ['terraform', 'devops', 'iac', 'roadmap']
---

# Terraform Module Inputs and Outputs

## Summary
Terraform modules use **Input Variables** and **Output Values** to create a stable interface for infrastructure components. Inputs allow modules to be parameterized and reusable across different environments, while Outputs expose specific resource attributes (like IP addresses or IDs) to other parts of the configuration or to the end user. Together, they enable **module composition**, allowing complex systems to be built from smaller, manageable units.

## Detailed Explanation

### 1. Input Variables
Input variables serve as parameters for a Terraform module. They allow you to customize the behavior of a module without modifying its source code.

#### Syntax and Arguments
```hcl
variable "instance_type" {
  type        = string
  description = "The size of the instance to deploy"
  default     = "t3.micro"
  
  # Validation ensures the input meets specific criteria
  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Only t3 series instances are allowed."
  }

  # Sensitive prevents the value from being printed in CLI logs
  sensitive = true
}
```

*   **Type Constraints**: Defines what kind of data is accepted (`string`, `number`, `bool`, `list`, `map`, `object`).
*   **Default**: Makes the variable optional. If omitted, the variable is required.
*   **Validation**: Custom rules to check the variable value during the `plan` phase.
*   **Sensitive**: Redacts the value in `terraform plan` and `apply` output to protect secrets.

### 2. Output Values
Outputs are the "return values" of a module. They expose information about the resources created within the module.

#### Syntax
```hcl
output "instance_public_ip" {
  value       = aws_instance.web.public_ip
  description = "The public IP address of the web server"
  
  # Protects sensitive data in CLI output
  sensitive = true
}
```

*   **Value**: The expression to be exported (usually a resource attribute).
*   **Sensitive**: If set to `true`, Terraform will hide the value in the console output (though it remains in the state file).

### 3. Module Composition and Communication
Module composition is the process of passing the output of one module into the input of another. This creates a data flow between isolated components.

#### Example: Passing Data
```hcl
# 1. Child Module (vpc/main.tf) defines an output
output "vpc_id" {
  value = aws_vpc.main.id
}

# 2. Parent Module (main.tf) calls child modules
module "network" {
  source = "./modules/vpc"
}

module "app_server" {
  source = "./modules/ec2"
  
  # Pass the output from the 'network' module as an input
  vpc_id = module.network.vpc_id
}
```

### 4. Best Practices
*   **Document everything**: Always provide a `description` for every input and output.
*   **Use explicit types**: Avoid using `type = any`. Be specific to catch errors early.
*   **Fail Fast with Validation**: Use `validation` blocks to ensure inputs are correct before attempting to create resources.
*   **Minimize Outputs**: Only export what is necessary for other modules or for verification.
*   **Naming Conventions**: Use consistent naming (e.g., `vpc_id` instead of `id_of_vpc`).

## Go (Golang) Context: Terratest

Go developers often interact with Terraform modules through **Terratest**, a Go library that makes it easy to write automated tests for infrastructure.

### Example: Testing Inputs and Outputs
In Terratest, you pass inputs via the `Vars` map and retrieve outputs using helper functions like `terraform.Output`.

```go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformModule(t *testing.T) {
	terraformOptions := &terraform.Options{
		// The path to where your Terraform code is located
		TerraformDir: "../modules/web_server",

		// Variables to pass to our Terraform code using -var options
		Vars: map[string]interface{}{
			"instance_name": "terratest-example",
			"instance_type": "t3.micro",
		},
	}

	// At the end of the test, run 'terraform destroy'
	defer terraform.Destroy(t, terraformOptions)

	// Run 'terraform init' and 'terraform apply'
	terraform.InitAndApply(t, terraformOptions)

	// Run 'terraform output' to get the value of an output variable
	publicIp := terraform.Output(t, terraformOptions, "public_ip")

	// Verify we received a valid IP
	assert.NotEmpty(t, publicIp)
}
```

## Interview Questions

*   **Q: How do you pass data between two different modules?**
    *   **A:** You define an `output` in the first module and then pass that output as an argument (input variable) to the `module` block of the second module.
*   **Q: What is the difference between a required and an optional input variable?**
    *   **A:** An optional variable has a `default` value defined in its `variable` block. A required variable does not have a `default` and must be provided by the caller.
*   **Q: How does the `sensitive = true` argument protect data?**
    *   **A:** It prevents the value from being displayed in clear text in the `terraform plan` and `apply` console output. However, the value is still stored in the state file in plain text (unless using a remote backend with encryption).
*   **Q: When would you use a `validation` block in a variable?**
    *   **A:** To enforce constraints on the input data (e.g., checking if a string matches a regex or if a number is within a specific range) to prevent invalid configurations from being applied.
