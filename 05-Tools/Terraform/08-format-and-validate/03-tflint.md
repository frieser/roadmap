---
---

## Summary

**TFLint** is a pluggable linter for Terraform that finds issues not caught by `terraform validate`. While the built-in validation checks for syntax and internal consistency, TFLint performs deep analysis on provider-specific resources to identify invalid instance types, deprecated attributes, and potential logic errors. Its pluggable architecture allows for specialized rulesets for AWS, GCP, Azure, and custom organizational standards.

## Detailed Explanation

### What is TFLint?

TFLint is a static analysis tool designed to improve the reliability and security of Terraform configurations. It acts as a second layer of defense after `terraform validate`. It is "pluggable," meaning you can enable or disable specific rulesets depending on your cloud provider and project needs.

### Key Features

*   **Pluggable Rules**: Supports specific plugins for AWS, Azure, GCP, and more.
*   **Provider-Specific Validation**: Catches errors like invalid EC2 instance types or invalid S3 bucket names before deployment.
*   **Custom Rules**: Allows organizations to enforce their own best practices (e.g., "All buckets must have encryption enabled").
*   **CI/CD Friendly**: Provides machine-readable output (JSON, JUnit) for automated pipelines.
*   **Deep Inspection**: Analyzes expressions and variables to detect issues that only emerge during plan/apply phases.

### Installation and Basic Usage

#### Installation

```bash
# Via Curl (Linux/macOS)
curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash

# Via Homebrew (macOS)
brew install tflint
```

#### Basic Commands

```bash
# Initialize TFLint (installs plugins defined in .tflint.hcl)
tflint --init

# Run the linter
tflint

# Run with a specific format
tflint --format compact
```

### Configuration File (.tflint.hcl)

The `.tflint.hcl` file manages plugins and rule configurations.

```hcl
config {
  format = "compact"
  module = true
  force = false
  disabled_by_default = false
}

# Example: AWS Plugin
plugin "aws" {
  enabled = true
  version = "0.21.1"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

# Explicitly disable or modify rules
rule "aws_instance_invalid_type" {
  enabled = true
}

rule "aws_s3_bucket_example_lifecycle_rule" {
  enabled = false
}
```

### TFLint vs. `terraform validate`

| Feature | `terraform validate` | TFLint |
| :--- | :--- | :--- |
| **Syntax Check** | Yes | Yes (delegates to HCL parser) |
| **Consistency** | Internal consistency only | Provider-specific logic |
| **Cloud Provider Awareness** | No | Yes (via plugins) |
| **Best Practices** | Minimal | High (naming, deprecated args) |
| **Unused Variables** | No | Yes |

### CI/CD Integration (GitHub Actions)

Integrating TFLint into a Go-based Terraform project pipeline:

```yaml
name: Lint
on: [push, pull_request]

jobs:
  tflint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup TFLint
        uses: terraform-linters/setup-tflint@v4
        with:
          tflint_version: latest

      - name: Show TFLint version
        run: tflint --version

      - name: Init TFLint
        run: tflint --init

      - name: Run TFLint
        run: tflint -f compact
```

### Writing Custom Rules in Go

TFLint rules are written in Go using the `tflint-plugin-sdk`. This allows for highly complex logic that isn't possible with simple regex-based linters.

#### Custom Rule Implementation Example

This Go code shows how to implement a rule that checks if an `aws_instance` has a specific tag.

```go
package rules

import (
	"fmt"
	"github.com/terraform-linters/tflint-plugin-sdk/hclext"
	"github.com/terraform-linters/tflint-plugin-sdk/tflint"
)

// InstanceTagRule checks if 'Environment' tag is present
type InstanceTagRule struct {
	tflint.DefaultRule
}

func NewInstanceTagRule() *InstanceTagRule {
	return &InstanceTagRule{}
}

func (r *InstanceTagRule) Name() string {
	return "aws_instance_environment_tag"
}

func (r *InstanceTagRule) Enabled() bool {
	return true
}

func (r *InstanceTagRule) Severity() tflint.Severity {
	return tflint.ERROR
}

func (r *InstanceTagRule) Check(runner tflint.Runner) error {
	// Define the schema to look for in the Terraform code
	resourceType := "aws_instance"
	schema := &hclext.BodySchema{
		Attributes: []hclext.AttributeSchema{
			{Name: "tags"},
		},
	}

	// Fetch all aws_instance resources
	resources, err := runner.GetResourceContent(resourceType, schema, nil)
	if err != nil {
		return err
	}

	for _, resource := range resources.Blocks {
		tagsAttr, exists := resource.Body.Attributes["tags"]
		if !exists {
			return runner.EmitIssue(r, "Missing 'tags' attribute", resource.DefRange)
		}

		// Complex logic can be implemented here to evaluate the tags map
		fmt.Printf("Analyzing resource: %s\n", resource.Labels[1])
	}

	return nil
}
```

#### Plugin Entry Point (`main.go`)

```go
package main

import (
	"github.com/terraform-linters/tflint-plugin-sdk/plugin"
	"github.com/terraform-linters/tflint-plugin-sdk/tflint"
	"github.com/youruser/tflint-ruleset-custom/rules"
)

func main() {
	plugin.Serve(&plugin.ServeOpts{
		RuleSet: &tflint.BuiltinRuleSet{
			Name:    "custom-rules",
			Version: "0.1.0",
			Rules: []tflint.Rule{
				rules.NewInstanceTagRule(),
			},
		},
	})
}
```

## Interview Questions

**Q: What is the primary advantage of using TFLint over the native `terraform validate` command?**
**A:** `terraform validate` only checks for syntax and basic internal consistency (e.g., missing variables). TFLint is cloud-aware; it uses plugins to check if the values you provide are valid for the specific provider (e.g., checking if an `instance_type` actually exists in AWS).

**Q: How do you handle "false positives" in TFLint?**
**A:** You can disable specific rules globally in the `.tflint.hcl` file, or you can use "annotations" in your Terraform code (e.g., `# tflint-ignore: aws_instance_invalid_type`) to ignore a specific line or block.

**Q: Why is TFLint considered "pluggable"?**
**A:** It uses a plugin architecture where rulesets for different providers (AWS, Azure, GCP) are maintained as separate binaries. This keeps the core TFLint engine lightweight and allows users to only install the rules they need.

**Q: How does TFLint integrate into a CI/CD pipeline?**
**A:** TFLint can be run as a step in a pipeline (like GitHub Actions or GitLab CI) after `terraform init` but before `terraform plan`. It can be configured to return a non-zero exit code on errors, failing the build if linting rules are violated.

**Q: What language is used to write custom TFLint rules, and what is the SDK called?**
**A:** Custom rules are written in **Go (Golang)** using the **tflint-plugin-sdk**. This allows developers to use the full power of Go to analyze HCL (HashiCorp Configuration Language) ASTs.

**Q: How do you update TFLint plugins to their latest versions?**
**A:** You update the version string in the `plugin` block within `.tflint.hcl` and then run `tflint --init` to download and install the new versions.

**Q: Can TFLint catch unused variables?**
**A:** Yes, there are built-in rules (and provider rules) that can detect variables, locals, and modules that are declared but never referenced in the configuration.
