---
tags: ['tools', 'roadmap', 'terraform', 'iac']
---

## Summary
`terraform fmt` is a CLI command that automatically rewrites Terraform configuration files (`.tf`) to a canonical format and style. Inspired by Go's `gofmt`, it ensures consistency across a codebase by enforcing standard indentation (2 spaces) and alignment, making diffs easier to read and collaboration smoother.

## Detailed Explanation

### What is `terraform fmt`?
The `terraform fmt` command applies a subset of the Terraform language style conventions to configuration files. It is opinionated and non-configurable, which eliminates debates about code style within a team. It modifies files in place by default.

Key characteristics:
- **Indentation**: Enforces 2-space indentation.
- **Alignment**: Aligns argument values (e.g., `=` signs) for better readability.
- **Spacing**: Fixes whitespace around operators and brackets.

### Usage and Flags

The basic usage scans the current directory for `.tf` files and formats them:

```bash
terraform fmt
```

Important flags:
- `-recursive`: Also processes files in subdirectories. Essential for monorepos or module structures.
- `-check`: Checks if the input is formatted. Returns exit status `0` if formatted, `3` if not. Used in CI/CD pipelines.
- `-diff`: Displays the differences between the original and formatted file without modifying it (unless used with other flags that imply modification, though often used with `-check` in CI to show *what* is wrong).

### Workflow Integration
1.  **Local Development**: Run `terraform fmt -recursive` before committing changes.
2.  **Pre-commit Hooks**: Use tools like `pre-commit` to automatically run formatting before a commit is allowed.
3.  **CI/CD**: Add a step in your pipeline to verify formatting:
    ```bash
    terraform fmt -check -recursive -diff
    ```
    This fails the build if code is not formatted, ensuring the canonical style is maintained in the shared repository.

### Go Application

`terraform fmt` was directly inspired by **`gofmt`**, a tool for the Go programming language (which Terraform is written in). Both tools share the philosophy that "gofmt's style is no one's favorite, yet gofmt is everyone's favorite" because it removes style friction.

#### Verifying Formatting with Terratest (Go)

When testing Terraform infrastructure using **Terratest** (a Go library), you can enforce formatting checks as part of your test suite.

```go
package test

import (
	"testing"

	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformFmt(t *testing.T) {
	t.Parallel()

	// Define options for running Terraform commands
	terraformOptions := &terraform.Options{
		TerraformDir: "../examples/basic",
	}

	// Clean up after the test (though fmt doesn't create resources)
	defer terraform.Destroy(t, terraformOptions)

	// Run 'terraform fmt -check' to verify formatting
	// We use RunTerraformCommand directly to execute 'fmt'
	// If the command fails (exit code 3), the test will fail
	output, err := terraform.RunTerraformCommandE(t, terraformOptions, "fmt", "-check", "-diff")

	// Assert that there was no error (meaning exit code 0, formatted correctly)
	assert.NoError(t, err, "Terraform code is not formatted. Run 'terraform fmt' to fix.")
}
```

This Go test ensures that the Terraform code in the specified directory adheres to the canonical format, catching style violations during the testing phase.

## Interview Questions

### Q: What is the purpose of `terraform fmt` and why should you use it?
**A:** `terraform fmt` rewrites Terraform configuration files to a canonical format and style. You should use it to ensure consistency across the codebase, improve readability, and reduce "noise" in git diffs caused by whitespace changes, making code reviews more effective.

### Q: How can you enforce Terraform code formatting in a CI/CD pipeline?
**A:** You can run `terraform fmt -check -recursive -diff` in your CI pipeline. The `-check` flag returns a non-zero exit code if files are not formatted, failing the build. The `-diff` flag prints the discrepancies to the logs so the developer knows what to fix.

### Q: Does `terraform fmt` validate the syntax or logic of the configuration?
**A:** No. `terraform fmt` only concerns itself with **style** (whitespace, alignment, indentation). To validate syntax and internal consistency (like missing required arguments), you must use `terraform validate`.
