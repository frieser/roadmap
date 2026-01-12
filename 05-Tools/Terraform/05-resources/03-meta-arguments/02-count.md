---
tags: ['tools', 'roadmap', 'terraform']
---

## Summary

The `count` meta-argument in Terraform allows you to create multiple instances of a resource or module from a single configuration block. It accepts a whole number, creating that many instances indexed from `0`. While powerful for identical replicas or conditional creation (0 or 1), it relies on numerical indexing, which can cause destructive recreations if the order of items changes, making `for_each` a better alternative for distinct items.

## Detailed Explanation

### What is `count`?

In Terraform, a `resource` block usually defines a single infrastructure object. The `count` meta-argument changes this behavior, looping over the block `N` times to create `N` distinct objects.

When `count` is set, Terraform distinguishes the instances by appending an integer index to the resource address (e.g., `aws_instance.server[0]`, `aws_instance.server[1]`).

### Basic Syntax

You define `count` with an integer value. Within the block, the `count` object is available, exposing `count.index` (the current iteration index).

```hcl
resource "aws_instance" "server" {
  count = 3 # Create 3 identical instances

  ami           = "ami-12345678"
  instance_type = "t3.micro"

  tags = {
    # Use count.index to create unique names: server-0, server-1, server-2
    Name = "server-${count.index}"
  }
}
```

### Conditional Creation

A common pattern for `count` is to toggle a resource on or off based on a boolean variable. Since Terraform doesn't have `if` blocks for resources, setting `count` to `0` effectively disables the resource.

```hcl
variable "enable_logging" {
  type    = bool
  default = false
}

resource "aws_s3_bucket" "logs" {
  # If true, create 1 bucket. If false, create 0.
  count = var.enable_logging ? 1 : 0

  bucket = "my-app-logs"
}
```

### Referencing Instances

When using `count`, the resource identifier refers to the **list** of instances, not a single object.

*   **Single Instance**: `aws_instance.server[0].id`
*   **All Attributes (Splat)**: `aws_instance.server[*].id` (returns a list of IDs)

### The "Index Shifting" Pitfall

The most critical limitation of `count` arises when using it to iterate over a list of **distinct** items (e.g., a list of usernames). Terraform tracks these resources *only* by their index (`user[0]`, `user[1]`), not by their content.

**The Problem:**
If you remove an item from the *middle* of the list, all subsequent items shift their index down by one. Terraform sees this as:
1.  "User 1 changed identity" (destroy/recreate)
2.  "User 2 changed identity" (destroy/recreate)
3.  "User N deleted"

**Example:**
List: `["Alice", "Bob", "Charlie"]`
Resources: `user[0]=Alice`, `user[1]=Bob`, `user[2]=Charlie`

*Action: Remove "Bob".*
New List: `["Alice", "Charlie"]`

Terraform Plan:
*   `user[0]` (Alice): No change.
*   `user[1]` (was Bob, now Charlie): **UPDATE/RECREATE** in place.
*   `user[2]` (was Charlie): **DESTROY**.

This is disastrous for stateful resources like databases. **Solution**: Use `for_each` for sets/maps of distinct items.

### Comparison: `count` vs `for_each`

| Feature | `count` | `for_each` |
| :--- | :--- | :--- |
| **Input** | Integer (`3`) | Map or Set of Strings |
| **Identifier** | Index (`[0]`, `[1]`) | Key (`["alice"]`, `["bob"]`) |
| **Stability** | Unstable (sensitive to order) | Stable (sensitive to keys only) |
| **Best For** | Identical replicas, Conditional (0/1) | Distinct items, Dynamic lists |

## Go Application (Terratest)

When writing automated tests for Terraform modules using **Go** (e.g., with [Terratest](https://terratest.gruntwork.io/)), you verify `count` logic by checking the length of the output list or validating specific indexed resources.

```go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformCount(t *testing.T) {
	t.Parallel()

	terraformOptions := &terraform.Options{
		TerraformDir: "../examples/terraform-count-example",
		Vars: map[string]interface{}{
			"instance_count": 3,
		},
	}

	// Clean up after test
	defer terraform.Destroy(t, terraformOptions)

	// Init and Apply
	terraform.InitAndApply(t, terraformOptions)

	// Validate Output: Expecting a list of 3 IDs
	// In HCL: output "instance_ids" { value = aws_instance.server[*].id }
	instanceIDs := terraform.OutputList(t, terraformOptions, "instance_ids")

	// Verify we got exactly 3 instances
	assert.Equal(t, 3, len(instanceIDs))

	// Verify the first ID is not empty
	assert.NotEmpty(t, instanceIDs[0])
}
```

## Interview Questions

**Q: When should you use `count` versus `for_each`?**
**A:** Use `count` when you need a specific number of *identical* resources (e.g., "5 worker nodes") or for conditional creation (0 or 1). Use `for_each` when you are creating resources based on a list or map of *distinct* items (e.g., "users", "subnets") where the identity of the resource depends on the map key, ensuring stability if the list order changes.

**Q: How do you access the ID of the third instance created by a resource using `count`?**
**A:** You access it using the index syntax: `aws_instance.web[2].id`. Note that indices are zero-based, so `[2]` refers to the third instance.

**Q: What happens if you remove the first element of a list used in a `count` loop?**
**A:** Terraform will shift all subsequent resources down by one index. It will likely update or destroy/recreate every single resource because their state ID (the index) no longer matches the configuration intended for that slot. This can cause unintentional data loss or downtime.

**Q: How can you conditionally create a resource using `count`?**
**A:** You can use a ternary operator: `count = var.enabled ? 1 : 0`. If `var.enabled` is true, count is 1 (resource created); if false, count is 0 (resource not created).
