---
tags: ['tools', 'roadmap', 'terraform', 'golang']
---

# Terraform State Replace Provider

## Summary
The `terraform state replace-provider` command is a state management utility used to update the source (Fully Qualified Name) of a provider for all resources in a Terraform state file. It allows for seamless migration between different providers (e.g., from a community fork to an official version or from a public registry to a private one) without requiring the destruction and recreation of managed infrastructure.

## Detailed Explanation

### What is `replace-provider`?
In Terraform, every resource in the state file is associated with a specific provider. This provider is identified by its **Fully Qualified Name (FQN)**, typically in the format `registry.terraform.io/namespace/name`. 

The `replace-provider` command updates these associations in bulk. It is particularly useful when the underlying provider implementation changes but the resource schema remains compatible.

### Why use it?
1.  **Provider Migration**: Moving from a legacy provider (e.g., unqualified providers like `-/aws`) to a modern qualified provider (`hashicorp/aws`).
2.  **Using Forks**: Switching to a custom fork of a provider for bug fixes or internal features (e.g., switching from `hashicorp/aws` to `registry.acme.corp/acme/aws`).
3.  **Registry Migration**: Moving providers from the public Terraform Registry to a private registry for security or compliance.
4.  **Avoid Infrastructure Churn**: Without this command, changing a provider in the configuration would cause Terraform to see the resources as "lost" (managed by a different provider), leading to their destruction and recreation.

### How to use it
The basic syntax is:
```bash
terraform state replace-provider [options] FROM_PROVIDER_FQN TO_PROVIDER_FQN
```

**Example:**
Migrating from an unqualified legacy provider to the official HashiCorp provider:
```bash
terraform state replace-provider "registry.terraform.io/-/aws" "registry.terraform.io/hashicorp/aws"
```

### Options
*   `-auto-approve`: Skips the interactive confirmation prompt.
*   `-lock=false`: Disables state locking (not recommended for remote backends).
*   `-lock-timeout`: Sets a duration to wait for a lock before failing.

## Go Context

In the Go ecosystem, Terraform state manipulation is often handled programmatically in automation tools, CI/CD pipelines, or custom infrastructure managers. The most common way to interact with Terraform from Go is through the `terraform-exec` library.

### Using `terraform-exec` in Go
While `terraform-exec` focuses on core commands like `plan` and `apply`, specialized state operations might require direct CLI execution or using community libraries like `tfmigrate`.

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os/exec"

	"github.com/hashicorp/terraform-exec/tfexec"
)

func main() {
	workingDir := "./infra"
	tfPath := "/usr/local/bin/terraform"

	tf, err := tfexec.NewTerraform(workingDir, tfPath)
	if err != nil {
		log.Fatalf("error running NewTerraform: %s", err)
	}

	// Example: Manual execution of replace-provider via Go's exec.Command
	// as high-level SDK support for all state subcommands can vary.
	ctx := context.Background()
	fromProvider := "registry.terraform.io/-/null"
	toProvider := "registry.terraform.io/hashicorp/null"

	cmd := exec.CommandContext(ctx, tfPath, "state", "replace-provider", "-auto-approve", fromProvider, toProvider)
	cmd.Dir = workingDir

	output, err := cmd.CombinedOutput()
	if err != nil {
		fmt.Printf("Error: %v\nOutput: %s\n", err, string(output))
		return
	}

	fmt.Println("Provider replaced successfully!")
}
```

### Terratest Integration
When writing infrastructure tests in Go with **Terratest**, you might need to verify that a provider migration doesn't break existing state.

```go
func TestProviderMigration(t *testing.T) {
	opts := &terraform.Options{
		TerraformDir: "../examples/migration-test",
	}

	// 1. Initial Apply with old provider
	terraform.InitAndApply(t, opts)

	// 2. Perform replacement (e.g., via shell command)
	shell.RunCommand(t, shell.Command{
		Command: "terraform",
		Args:    []string{"state", "replace-provider", "-auto-approve", "old/provider", "new/provider"},
		Dir:     opts.TerraformDir,
	})

	// 3. Verify no changes are pending (Idempotency check)
	plan := terraform.InitAndPlan(t, opts)
	assert.Contains(t, plan, "No changes. Your infrastructure matches the configuration.")
}
```

## Interview Questions

**Q: What is the main risk when running `terraform state replace-provider`?**
**A:** The main risk is corrupting the state file or misaligning resources with a provider that doesn't actually support them. Terraform always creates a mandatory backup of the state file before execution to mitigate this risk.

**Q: Can you use `replace-provider` to change a resource type?**
**A:** No. `replace-provider` only changes the source of the provider associated with the resource. If the resource type itself changes between providers (e.g., `aws_instance` vs `custom_instance`), you must use `terraform state mv` or `import`/`rm`.

**Q: Why is this command preferred over `terraform state rm` followed by `terraform import`?**
**A:** Efficiency and safety. `replace-provider` is an atomic, bulk operation. Doing it manually via `rm` and `import` for hundreds of resources is time-consuming, error-prone, and risks losing resource-specific metadata stored in the state.

**Q: Where does this fit in the Terraform lifecycle?**
**A:** It belongs to the **Refactoring and Maintenance** phase. It is typically performed during major version upgrades, infrastructure migrations, or when adopting a private registry.

## Roadmap Placement
In the **roadmap.sh Terraform Roadmap**, this topic fits under:
*   **State Management**
    *   **Inspecting and Modifying State**
        *   Advanced State Commands (`mv`, `rm`, `replace-provider`)
