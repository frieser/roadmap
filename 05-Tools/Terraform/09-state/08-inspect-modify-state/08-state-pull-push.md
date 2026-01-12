---
tags: ['tools', 'roadmap', 'terraform']
---

# Terraform State: Pull and Push

## Summary
The `terraform state pull` and `terraform state push` commands are low-level tools used to manually transfer state data between a local environment and a remote backend. `pull` downloads the current state from the backend to `stdout`, while `push` uploads a local state file to the remote backend, potentially overwriting existing data. These are critical for manual state recovery, migration, and troubleshooting.

## Detailed Explanation

### 1. terraform state pull
This command downloads the state from the currently configured backend and outputs the raw JSON to standard output (`stdout`).

*   **Primary Use Cases**:
    *   **Inspection**: Reading the raw state to understand how Terraform sees a resource without a local `.tfstate` file.
    *   **Manual Editing**: Exporting state to a file, making manual JSON corrections (like fixing a corrupted resource attribute), and preparing it for a re-upload.
    *   **Migration**: Backing up state before changing the `backend` block in configuration.
*   **Syntax**:
    ```bash
    terraform state pull > backup.tfstate
    ```

### 2. terraform state push
This command uploads a specified local state file to the remote backend. It is considered a dangerous operation because it can overwrite the source of truth for your infrastructure.

*   **Primary Use Cases**:
    *   **Disaster Recovery**: Restoring a known good state from a backup after a failed operation or accidental deletion.
    *   **Applying Manual Fixes**: Pushing back a modified state file that was previously "pulled" and edited.
    *   **Manual Migration**: Moving state between different organizations or backend types where `terraform init -migrate-state` might fail.
*   **Safety**: Terraform performs a lineage and serial check. If the local state has a different history, it will fail unless the `-force` flag is used.
*   **Syntax**:
    ```bash
    terraform state push modified.tfstate
    ```

### 3. State Locking
Even though these are low-level commands, Terraform still respects (and attempts to acquire) state locks if the backend supports them (e.g., DynamoDB for S3). This prevents race conditions during manual state manipulation.

## Go Application

Terraform is built in Go, and developers often use its internal libraries to build custom tooling for state analysis or automation. The following example demonstrates how to use the `states/statefile` package to read a Terraform state file programmatically.

```go
package main

import (
	"fmt"
	"os"

	"github.com/hashicorp/terraform/states/statefile"
)

func main() {
	// Open a local state file (previously pulled via 'terraform state pull')
	f, err := os.Open("terraform.tfstate")
	if err != nil {
		panic(err)
	}
	defer f.Close()

	// Parse the state file
	file, err := statefile.Read(f)
	if err != nil {
		fmt.Printf("Error reading state: %v\n", err)
		return
	}

	// Accessing metadata
	fmt.Printf("Terraform Version: %s\n", file.TerraformVersion)
	fmt.Printf("Serial: %d\n", file.Serial)

	// Iterate through resources in the state
	for _, res := range file.State.Modules[""].Resources {
		fmt.Printf("Resource found: %s.%s\n", res.Addr.Resource.Type, res.Addr.Resource.Name)
	}
}
```

## Interview Questions

**Q: When would you use `terraform state pull` instead of just opening the `terraform.tfstate` file?**
**A:** When using a **remote backend** (like S3, GCS, or Azure Blob), there is no local `terraform.tfstate` file on your machine. The `pull` command is the only way to retrieve the current snapshot of the infrastructure from the remote storage for inspection or manual modification.

**Q: What is the risk of using `terraform state push -force`?**
**A:** The `-force` flag bypasses safety checks for **lineage** (unique ID of the state history) and **serial number** (version increment). Using it can result in losing all history of the infrastructure, potentially causing Terraform to "forget" existing resources, leading to duplicate creation or orphan resources in the cloud provider.

**Q: How do you migrate state from a local backend to an S3 backend manually using these commands?**
**A:** 
1. Run `terraform state pull > local.tfstate` to ensure you have the latest state.
2. Update the `backend "s3" {}` block in your Terraform configuration.
3. Run `terraform init`.
4. Use `terraform state push local.tfstate` to upload the local data to S3. 
*(Note: `terraform init -migrate-state` is the preferred automated way, but push/pull is the manual fallback).*
