---
tags: ['tools', 'roadmap']
---

## Summary
Terraform provisioners are used to execute scripts on a local or remote machine as part of resource creation or destruction. However, HashiCorp explicitly recommends using them only as a **last resort**. They should be avoided because they break the declarative nature of Terraform, are not tracked in the state file like regular resources, and often lead to brittle infrastructure.

## Detailed Explanation

### Why Provisioners are a "Last Resort"
Terraform is designed to be a **declarative** tool, meaning you describe the desired state, and Terraform handles the "how". Provisioners introduce **imperative** logic, which brings several challenges:
1. **No State Tracking**: Terraform doesn't track the actions of a provisioner. If a script modifies a file, Terraform doesn't know about it and cannot undo it or detect drift.
2. **Brittle Connections**: Provisioners require direct network access (SSH/WinRM) to the target machine, which complicates security groups and firewall rules.
3. **Error Handling**: If a provisioner fails, the resource is marked as "tainted" and will be destroyed and recreated on the next apply, which might not be desirable.

### Better Alternatives
Before reaching for provisioners, consider these purpose-built solutions:
- **Cloud-Init / user_data**: Most cloud providers support `user_data` to run scripts during the first boot. This is more reliable and handled by the provider.
- **Custom Images**: Use tools like **Packer** to build pre-configured images (AMIs, VHDs) that already contain the necessary software.
- **Configuration Management**: Use tools like **Ansible**, **Chef**, or **Puppet** which are designed for server configuration.
- **Post-Provisioning Handlers**: Use systemd units or Kubernetes Init Containers for application-level setup.

### When to Use Provisioners
You might still use provisioners if:
- You need to perform a cleanup action upon destruction that no other tool supports.
- You are working with legacy systems that do not support `cloud-init`.
- You need to bootstrap a configuration management agent (like Ansible) on a fresh node.

### HCL Example: local-exec
The `local-exec` provisioner invokes a local executable after a resource is created.

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  # local-exec runs on the machine running Terraform
  provisioner "local-exec" {
    command = "echo ${self.private_ip} >> private_ips.txt"
  }
}
```

## Interview Questions

**Q: Why does HashiCorp recommend using provisioners as a last resort?**
**A:** Provisioners are imperative and break the declarative model of Terraform. They aren't tracked in the state, making it impossible for Terraform to manage or detect drift in the changes they make. They also require complex connectivity (SSH/WinRM) and make error recovery difficult by tainting resources.

**Q: What happens to a resource if its creation-time provisioner fails?**
**A:** If a creation-time provisioner fails, Terraform marks the resource as **tainted**. This means that although the resource exists, it is considered "unclean." On the next `terraform apply`, Terraform will destroy the tainted resource and attempt to recreate it.

**Q: Mention three alternatives to using Terraform provisioners.**
**A:** 1. Using `user_data` or `cloud-init` for initial bootstrapping. 2. Building custom machine images using Packer. 3. Using configuration management tools like Ansible to manage the server state after it is provisioned.
