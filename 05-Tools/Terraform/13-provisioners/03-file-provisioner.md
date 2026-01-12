---
tags: ['tools', 'roadmap']
---

## Summary
The `file` provisioner is used to upload files or directories from the machine running Terraform to a newly created remote resource. It requires a valid `connection` (SSH or WinRM) to the target machine and is commonly used to transfer configuration files or installation scripts before executing them with `remote-exec`.

## Detailed Explanation

### How the File Provisioner Works
The `file` provisioner copies data from the **source** (local) to the **destination** (remote). It supports:
- **Single files**: Copying a specific configuration file.
- **Directories**: Copying an entire folder of scripts or assets.
- **Raw Content**: Using the `content` attribute to write a string directly to a remote file instead of pointing to a local source.

### Connectivity Requirements
To use the `file` provisioner, you must define a `connection` block. This block tells Terraform how to communicate with the remote resource:
- **Type**: `ssh` (Linux) or `winrm` (Windows).
- **Credentials**: Username, password, or private key.
- **Host**: The IP address or DNS name (often retrieved via `self.public_ip`).

### Best Practices and Limitations
1. **Permissions**: The destination path must be writable by the user specified in the `connection` block. 
2. **Root Access**: You cannot directly upload to locations requiring `sudo` (like `/etc/`). The standard workaround is to upload to `/tmp/` and then use a `remote-exec` provisioner with `sudo mv` to move the file to its final destination.
3. **Security**: Hardcoding private keys in the `connection` block is a security risk. Use variables or an SSH agent instead.

### HCL Example: File Provisioner

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  # Connection block is mandatory for file and remote-exec
  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }

  # Copy a local file to the remote machine
  provisioner "file" {
    source      = "conf/nginx.conf"
    destination = "/tmp/nginx.conf"
  }

  # Create a file from a string
  provisioner "file" {
    content     = "Server Name: ${self.tags.Name}"
    destination = "/tmp/server_info.txt"
  }
}
```

## Interview Questions

**Q: What are the two mandatory arguments for a `file` provisioner block?**
**A:** The `source` (or `content`) and the `destination` arguments. `source` is the path to the local file/directory, while `destination` is the absolute path on the remote machine.

**Q: Why do we often upload files to `/tmp` instead of their final location?**
**A:** Provisioners run as the user defined in the `connection` block, who usually doesn't have root permissions. Since you cannot run a `file` provisioner with `sudo`, you upload to a world-writable directory like `/tmp` and then use `remote-exec` with `sudo` to move it.

**Q: Is a `connection` block required for the `file` provisioner?**
**A:** Yes. Unlike the `local-exec` provisioner which runs on the machine executing Terraform, the `file` provisioner must connect to the remote resource via SSH or WinRM to transfer the data.
