---
tags: ['tools', 'roadmap', 'terraform', 'testing', 'go']
---

# Testing Modules in Terraform

## Summary
Testing Terraform modules is a critical practice for ensuring infrastructure stability, reliability, and reusability. It involves validating a module's logic and its ability to provision resources correctly across various configurations. The primary strategies include using an **examples/ directory** for isolated testing, leveraging **native Terraform testing** (`.tftest.hcl`), and using **Terratest** for programmatic, table-driven, and parallel validation.

## Detailed Explanation

### 1. The "examples/" Directory Pattern
A best practice for Terraform module development is to include an `examples/` directory at the root of the module.
*   **Purpose**: Each subdirectory in `examples/` represents a specific use case (e.g., `examples/basic`, `examples/complete-vpc`, `examples/internal-only`).
*   **Documentation**: Examples serve as executable documentation for users.
*   **Test Entry Point**: Tests (both native and Terratest) use these examples as the "root" configuration to initialize and apply, ensuring the module works in real-world scenarios.

### 2. Native Module Testing (Terraform v1.6+)
Terraform's built-in framework allows testing modules without external dependencies.
*   **Mocking (v1.7+)**: Essential for testing modules that interact with many data sources or providers without requiring real cloud credentials.
*   **Setup Modules**: `run` blocks can call alternate modules (e.g., a setup module to create a VPC before testing a database module).
*   **Validation**: Assertions can check outputs and resource attributes after a `plan` or `apply`.

### 3. Terratest for Modular Testing (Go)
Terratest provides the most flexibility for complex module testing:
*   **Table-Driven Tests**: A Go idiom where multiple test cases (different inputs) are defined in a slice and iterated over. This is perfect for testing a module with various variable combinations (e.g., testing a "Compute" module with different OS images or instance types).
*   **Parallel Testing**: By calling `t.Parallel()`, Go runs multiple tests simultaneously. In Terratest, this means multiple Terraform environments are provisioned in parallel, significantly reducing CI/CD time.
*   **Internal Module Patterns**: For large organizations with "internal" modules (not publicly shared), tests should be bundled within the same repository to ensure that any change to the module code is immediately verified by the associated tests.

---

## Go (Terratest) Code Example: Table-Driven & Parallel Testing

This example demonstrates testing a generic "Server" module with multiple configurations using a table-driven approach and parallel execution.

```go
package test

import (
	"fmt"
	"testing"

	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestServerModule(t *testing.T) {
	// Define the test cases
	testCases := []struct {
		name         string
		instanceType string
		environment  string
	}{
		{"SmallInstance", "t3.nano", "dev"},
		{"MediumInstance", "t3.medium", "staging"},
	}

	for _, tc := range testCases {
		// Capture range variable
		tc := tc

		t.Run(tc.name, func(t *testing.T) {
			// Mark test as parallel
			t.Parallel()

			terraformOptions := &terraform.Options{
				// Path to the example that uses the internal module
				TerraformDir: "../examples/server-example",

				// Variables to pass to the module
				Vars: map[string]interface{}{
					"instance_type": tc.instanceType,
					"env":           tc.environment,
				},

				// Handle common retryable errors
				RetryableTerraformErrors: map[string]string{
					".*": "Transient cloud error",
				},
				MaxRetries: 3,
			}

			// Clean up at the end of the test
			defer terraform.Destroy(t, terraformOptions)

			// Init and Apply
			terraform.InitAndApply(t, terraformOptions)

			// Assertions
			instanceID := terraform.Output(t, terraformOptions, "instance_id")
			assert.NotEmpty(t, instanceID)
			
			envOutput := terraform.Output(t, terraformOptions, "environment")
			assert.Equal(t, tc.environment, envOutput)
		})
	}
}
```

---

## Interview Questions

**Q: Why is the `examples/` directory considered a "Primary Source" for module testing?**
**A:** Because it provides a clean, user-facing way to exercise the module. By testing the examples, you ensure that the module's public interface (variables and outputs) works as documented and that common configurations are valid.

**Q: How does `t.Parallel()` in Go improve Terraform testing efficiency?**
**A:** It allows multiple independent Terraform tests to run at the same time. Since infrastructure provisioning is often the bottleneck (taking minutes), running 5-10 tests in parallel can reduce a 50-minute test suite to 10 minutes, assuming the cloud provider's rate limits allow it.

**Q: When should you use a "Setup Module" in a Terraform test?**
**A:** When the module under test has dependencies that are not part of its own scope. For example, if you are testing a Kubernetes application module, you might use a setup module in a `run` block to provision the EKS cluster first.

**Q: What is the benefit of Table-Driven tests for infrastructure modules?**
**A:** They allow you to test "matrix" configurations (e.g., 3 OS types x 2 Regions) with minimal code duplication. It ensures that the module logic holds true across different scales and environments.

**Q: How do you prevent "State Pollution" when running parallel tests for the same module?**
**A:** By ensuring each test run uses unique identifiers (e.g., `random_id` in Terraform or `random.UniqueId()` in Terratest) for resource names and using separate backends or temporary state files for each test instance.
