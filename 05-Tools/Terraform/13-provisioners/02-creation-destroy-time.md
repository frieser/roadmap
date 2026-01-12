---
tags: ['tools', 'roadmap']
---

## Summary
Terraform provisioners can be categorized into **creation-time** and **destroy-time** based on when they execute in the resource lifecycle. Creation-time provisioners (the default) run immediately after a resource is created, while destroy-time provisioners run before the resource is deleted.

## Detailed Explanation

### Creation-Time Provisioners
This is the default behavior. If you define a `provisioner` block without specifying `when = destroy`, it runs only during the **creation** of the resource.

- **Success**: If it completes successfully, the resource is recorded in the state.
- **Failure**: If it fails, Terraform taints the resource. It is marked as created but potentially non-functional. Terraform will attempt to destroy and recreate it during the next `apply`.

### Destroy-Time Provisioners
These are explicitly defined using the `when = destroy` argument. They run **before** the resource is actually deleted from the provider.

- **Purpose**: Primarily used for cleanup tasks, such as unregistering a node from a monitoring system, draining a database connection, or removing a license key.
- **Failure**: If a destroy-time provisioner fails, Terraform will stop and return an error. The resource will **not** be removed from the state, allowing you to fix the issue and retry.

### Configuration and Self Reference
Within a provisioner block, you can use the `self` object to refer to the resource's own attributes (like `self.private_ip`), which is useful since the resource might not be fully indexed in the global scope yet.

### HCL Example: Creation and Destroy Time

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  # Creation-time provisioner (Default)
  provisioner "local-exec" {
    command = "echo 'Instance ${self.id} created' > status.txt"
  }

  # Destroy-time provisioner
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Instance ${self.id} is being destroyed' >> status.txt"
  }
}
```

## Interview Questions

**Q: What is the difference between a creation-time and a destroy-time provisioner?**
**A:** A creation-time provisioner runs after the resource is created and is the default behavior. A destroy-time provisioner runs before the resource is deleted and must be explicitly configured with `when = destroy`.

**Q: How does Terraform handle a failure in a destroy-time provisioner?**
**A:** If a destroy-time provisioner fails, Terraform halts the destruction process and keeps the resource in the state file. This ensures the resource isn't deleted until the cleanup logic successfully runs.

**Q: Can you use variables from other resources inside a provisioner?**
**A:** Yes, but it is better to use the `self` object to refer to attributes of the resource the provisioner is defined in. This prevents circular dependencies and ensures you are using the correct instance data during the lifecycle event.
