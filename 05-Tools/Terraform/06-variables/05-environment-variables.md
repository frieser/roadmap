---
tags: ['terraform', 'iac', 'tools', 'roadmap']
---

# Terraform Environment Variables

## Summary

Terraform uses environment variables to configure its behavior, manage credentials, and set input variables. The most critical use case is the `TF_VAR_name` pattern, which allows passing values to root module variables without using variable files or CLI flags. Other environment variables like `TF_LOG` and `TF_CLI_ARGS` provide control over logging and default command-line arguments.

## Detailed Explanation

### 1. Setting Input Variables (TF_VAR_name)

Terraform automatically searches the environment for variables starting with `TF_VAR_`. The suffix determines the name of the variable in your HCL code.

- **Format**: `TF_VAR_<variable_name>`
- **Example**:
  ```bash
  export TF_VAR_region="us-east-1"
  export TF_VAR_instance_type="t3.micro"
  ```
  These will populate:
  ```hcl
  variable "region" { type = string }
  variable "instance_type" { type = string }
  ```

### 2. Common Terraform Environment Variables

| Variable | Description | Values |
|----------|-------------|--------|
| `TF_LOG` | Enables debug logs. | `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR` |
| `TF_LOG_PATH` | File path to save logs. | `/path/to/terraform.log` |
| `TF_INPUT` | Disables interactive prompts if set to `0` or `false`. | `0`, `false` |
| `TF_CLI_ARGS` | Additional arguments to append to all commands. | `-no-color`, `-input=false` |
| `TF_CLI_ARGS_name` | Arguments for a specific command (e.g., `TF_CLI_ARGS_plan`). | `-lock=false` |
| `TF_DATA_DIR` | Changes where Terraform keeps its data (default `.terraform`). | `/path/to/dir` |
| `TF_WORKSPACE` | Selects a specific workspace. | `prod`, `staging` |
| `TF_IN_AUTOMATION` | Adjusts output for CI environments. | `true`, `1` |

### 3. Variable Precedence (Lowest to Highest)

When the same variable is defined in multiple places, Terraform follows this order:

1. **Environment Variables (`TF_VAR_name`)** - *Lowest precedence*
2. `terraform.tfvars` or `terraform.tfvars.json`
3. `*.auto.tfvars` or `*.auto.tfvars.json` (lexical order)
4. Command-line flags (`-var` and `-var-file`) - *Highest precedence*

> [!NOTE]
> This means a `-var` flag will ALWAYS override an environment variable.

### 4. Security Implications

- **Secrets in Environment**: Environment variables are often visible in process lists (`ps aux`) and stored in shell history (`~/.bash_history`). 
- **CI/CD Logs**: Many CI systems print the environment by default. Ensure sensitive variables are marked as "masked" or "secret" in your CI provider.
- **State File**: Regardless of how a variable is passed (CLI, Env, or File), its value is stored in the **plain-text** state file unless using a remote backend with encryption. Mark variables as `sensitive = true` to prevent them from appearing in CLI output.

## Go Application (Terratest)

When writing infrastructure tests in Go using `terratest`, you can inject environment variables into the Terraform execution context using the `EnvVars` field in `terraform.Options`.

### Go Example using Terratest

```go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformEnvVarsExample(t *testing.T) {
	terraformOptions := &terraform.Options{
		// Path to the Terraform code
		TerraformDir: "../examples/terraform-basic-example",

		// Variables passed via -var flags (High Priority)
		Vars: map[string]interface{}{
			"instance_name": "terratest-example",
		},

		// Environment variables (TF_VAR_ pattern or provider credentials)
		EnvVars: map[string]string{
			"TF_VAR_region":      "us-east-1",
			"AWS_DEFAULT_REGION": "us-east-1",
			"TF_LOG":             "DEBUG",
		},
	}

	// Clean up resources at the end of the test
	defer terraform.Destroy(t, terraformOptions)

	// Run terraform init and apply
	terraform.InitAndApply(t, terraformOptions)

	// Validate output
	output := terraform.Output(t, terraformOptions, "instance_id")
	assert.NotEmpty(t, output)
}
```

## Go-Specific Applications

In Go-based CLI tools or operators (like Crossplane providers or custom Terraform wrappers), environment variables are typically managed via the `os` package:

```go
import "os"

func setTerraformVars() {
    os.Setenv("TF_VAR_db_password", "supersecret")
    os.Setenv("TF_IN_AUTOMATION", "true")
}
```

## Interview Preparation

**Q: In what order does Terraform load variables?**
**A:** Environment variables, then `terraform.tfvars`, then `*.auto.tfvars`, and finally CLI flags (`-var`).

**Q: How do you enable detailed logging for Terraform?**
**A:** Set `TF_LOG` to `TRACE` or `DEBUG`. You can optionally set `TF_LOG_PATH` to save the output to a file.

**Q: Why is it dangerous to pass secrets via TF_VAR environment variables in a shared shell?**
**A:** Because they can be leaked in the shell history or seen by other users on the system via `ps` commands.

**Q: How does `TF_IN_AUTOMATION` change Terraform's behavior?**
**A:** It reduces the verbosity of the output and removes suggestions about commands that are only relevant for interactive usage, making the logs cleaner for CI/CD.
