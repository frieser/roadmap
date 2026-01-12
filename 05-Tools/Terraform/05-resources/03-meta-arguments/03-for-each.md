---
tags: ['tools', 'roadmap', 'terraform', 'hcl']
---

## Summary
The `for_each` meta-argument is a powerful Terraform feature that creates multiple instances of a resource or module from a map or set of strings. Unlike `count`, which relies on numerical indices, `for_each` assigns a stable, unique key to each instance, ensuring that changes to the collection order do not trigger unnecessary resource recreation. It is the standard best practice for managing dynamic collections of resources where identity matters.

## Detailed Explanation

### Definition and Purpose
In Terraform, `for_each` allows you to iterate over a collection (map or set) and define a resource or module for each item. This avoids code duplication and enables dynamic infrastructure provisioning.

Its most significant advantage over the older `count` meta-argument is **stability**. Because `for_each` uses keys (strings) rather than indices (integers) to identify resources, adding or removing items from the middle of a list does not shift the identity of subsequent resources.

### Syntax and Usage

The `for_each` argument accepts a map or a set of strings.

#### 1. Using a Map
When using a map, the key becomes the identifier, and the value can be accessed within the resource using `each.value`.

```hcl
variable "subnets" {
  description = "Map of subnet names to CIDR blocks"
  type        = map(string)
  default = {
    "web" = "10.0.1.0/24"
    "app" = "10.0.2.0/24"
    "db"  = "10.0.3.0/24"
  }
}

resource "aws_subnet" "example" {
  for_each = var.subnets

  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
  tags = {
    Name = each.key
  }
}
```

#### 2. Using a Set of Strings
If you have a list, you must convert it to a set using `toset()`. This ensures uniqueness and allows Terraform to use the string values as keys.

```hcl
resource "aws_iam_user" "users" {
  for_each = toset(["alice", "bob", "charlie"])
  name     = each.key
}
```

### The `each` Object
Inside a block using `for_each`, a special object named `each` is available:
- **`each.key`**: The map key or the set element corresponding to the current instance.
- **`each.value`**: The map value corresponding to the current instance. (For sets, `each.value` is identical to `each.key`).

### Comparison: `for_each` vs `count`

| Feature | `count` | `for_each` |
| :--- | :--- | :--- |
| **Input Type** | Integer | Map or Set(String) |
| **Identifier** | Index (0, 1, 2...) | Key (String) |
| **Stability** | **Low**. Removing item at index 0 shifts all indices, forcing recreation of all subsequent resources. | **High**. Removing an item only affects that specific resource key. |
| **Best For** | Identical resources, simple toggles (0 or 1). | Distinct resources, dynamic collections. |

**Scenario**: You have users `["alice", "bob"]`.
- With `count`, removing "alice" makes "bob" index 0. Terraform destroys "alice" and *updates* the old index 0 resource to be "bob".
- With `for_each`, removing "alice" simply destroys the resource `iam_user["alice"]`. `iam_user["bob"]` is untouched.

### Advanced Usage: Dynamic Blocks
`for_each` is also used inside `dynamic` blocks to generate nested configuration blocks, such as security group rules.

```hcl
resource "aws_security_group" "example" {
  name = "example-sg"
  # ...

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidrs
    }
  }
}
```

### Go Application (Terratest)
When testing Terraform code that uses `for_each`, the output is typically a map. In **Go** (specifically with Terratest), you need to handle these outputs to verify your infrastructure.

**Terraform Output:**
```hcl
output "subnet_ids" {
  value = { for k, v in aws_subnet.example : k => v.id }
}
```

**Go Test (`main_test.go`):**
```go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformForEach(t *testing.T) {
	t.Parallel()

	terraformOptions := &terraform.Options{
		TerraformDir: "../examples/for-each-example",
        // Pass variables if needed
		Vars: map[string]interface{}{
			"subnets": map[string]string{
				"test-web": "10.0.10.0/24",
			},
		},
	}

	defer terraform.Destroy(t, terraformOptions)
	terraform.InitAndApply(t, terraformOptions)

	// Retrieve the output as a map
	// outputMap will be map[string]string
	outputMap := terraform.OutputMap(t, terraformOptions, "subnet_ids")

	// Verify we have the expected keys and values
	assert.Contains(t, outputMap, "test-web")
	assert.Regexp(t, "^subnet-[a-f0-9]+$", outputMap["test-web"])
}
```

## Interview Questions

**Q: Why is `for_each` preferred over `count` for creating resources from a list?**
**A:** `for_each` uses stable identifiers (keys) instead of numerical indices. This prevents the "index shifting" problem where removing an item from the middle of a list causes Terraform to recreate all subsequent resources.

**Q: Can you use `for_each` with a module?**
**A:** Yes, `for_each` supports modules (since Terraform 0.13), allowing you to instantiate multiple copies of a module configuration keyed by unique identifiers.

**Q: How do you handle a list of strings with `for_each`?**
**A:** You must convert the list to a set using `toset(var.list)`. `for_each` requires a set or map because it needs unique, unordered keys.

**Q: What is the limitation regarding computed values?**
**A:** The keys used in `for_each` must be known at `plan` time. You cannot use values that are only known after apply (computed values) as keys, although you can use them as values in the map.

**Q: What objects are available inside a `for_each` block?**
**A:** The `each` object, which contains `each.key` and `each.value`. For sets, these are identical.
