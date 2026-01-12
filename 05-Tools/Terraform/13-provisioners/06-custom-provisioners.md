---
tags: ['tools', 'roadmap', 'terraform']
---

# 06 - Provisioners Overview & Patterns

## Summary
Provisioners are a feature used to model specific actions on a local or remote machine to prepare resources for service. HashiCorp explicitly recommends using them as a **"last resort"** because they break the declarative nature of Terraform. Modern patterns favor `terraform_data` over the legacy `null_resource` for orchestrating these imperative steps.

## Detailed Explanation

### **Why are Provisioners a "Last Resort"?**
1. **Not Declarative**: They represent a sequence of actions, not a desired state.
2. **Lack of State Tracking**: Terraform doesn't know what the script actually did. If the script changes a file, Terraform won't detect it as a "drift".
3. **Fragility**: They depend on external factors like network connectivity, SSH access, and local dependencies.
4. **No Plan Insight**: `terraform plan` cannot predict the outcome of a provisioner.

### **The Modern Alternative: `terraform_data`**
Introduced in Terraform 1.4, `terraform_data` is a built-in resource that replaces the legacy `null_resource` from the `hashicorp/null` provider.
- **No Provider Required**: It's built into Terraform core.
- **Triggers**: It uses `triggers_replace` to force a recreation (and thus a re-run of provisioners) when a value changes.
- **Storage**: It can store arbitrary data in the state.

```hcl
resource "terraform_data" "bootstrap" {
  triggers_replace = [
    aws_instance.web.id,
    var.script_version
  ]

  provisioner "local-exec" {
    command = "echo 'Bootstrapped instance ${aws_instance.web.id}'"
  }
}
```

### **The Legacy: `null_resource`**
Before 1.4, `null_resource` was used to wrap provisioners that didn't belong to a specific resource. It required the `null` provider.
```hcl
resource "null_resource" "example" {
  triggers = {
    instance_ids = join(",", aws_instance.web.*.id)
  }

  provisioner "local-exec" {
    command = "echo Hello"
  }
}
```

### **Provisioners vs Modules/Providers**
- **Providers**: The primary way to manage resources. If a provider exists for your tool (e.g., Helm, Kubernetes, Ansible), use it instead of a provisioner.
- **Modules**: Used for packaging and reusability. A module can contain provisioners, but it's better if it contains resources managed by providers.

## Interview Questions

**Q: Why does HashiCorp recommend using provisioners only as a last resort?**
**A:** Because they are imperative, don't follow the declarative model, are not well-represented in `terraform plan`, and don't provide the same idempotency guarantees as native resources.

**Q: What is the purpose of `terraform_data`?**
**A:** It is a built-in resource used to store arbitrary data or to trigger provisioners without needing an external provider like `null`. It's the modern replacement for `null_resource`.

**Q: How do you force a provisioner to re-run without recreating the main resource?**
**A:** Wrap the provisioner in a `terraform_data` (or `null_resource`) and use a `trigger` (or `triggers_replace`) that changes when you want the re-run to occur.

**Q: What is the difference between a creation-time provisioner and a destroy-time provisioner?**
**A:** Creation-time runs after resource creation; if it fails, the resource is tainted. Destroy-time runs before the resource is deleted; if it fails, the destroy operation stops.

**Q: When should you use a provisioner instead of a dedicated provider?**
**A:** Only when no provider exists for the task, or when the task is a simple one-off script that doesn't justify the complexity of a custom provider or complex configuration management integration.
