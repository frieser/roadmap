---
---

## Summary
SSH (Secure Shell) is the standard protocol for secure remote login. `scp` and `rsync` leverage SSH to transfer files securely. `rsync` is generally preferred over `scp` for its efficiency (delta transfer) and versatility.

## Detailed Explanation

### SSH
*   **Connect**: `ssh user@host`.
*   **Keys**: Using `ssh-keygen` to create a keypair and `ssh-copy-id` to install the public key allows passwordless login.
*   **Config**: `~/.ssh/config` allows aliases (`ssh myserver`).

### SCP (Secure Copy)
*   **Copy to remote**: `scp file.txt user@host:/path/`.
*   **Copy from remote**: `scp user@host:/path/file.txt .`.
*   **Note**: SCP is simple but inefficient (copies everything blindly).

### Rsync (Remote Sync)
*   **Delta Sync**: Only sends differences between source and dest.
*   **Compression**: `-z` compresses data during transfer.
*   **Archive**: `-a` preserves permissions, times, and symlinks.
*   **Usage**: `rsync -avz local_dir/ user@host:/remote_dir/`.

## Go-Specific Context/Examples

Go's `golang.org/x/crypto/ssh` package allows you to build SSH clients and servers.

### Example: Remote Command via SSH in Go
```go
package main

import (
	"fmt"
	"golang.org/x/crypto/ssh"
)

func main() {
	config := &ssh.ClientConfig{
		User: "myuser",
		Auth: []ssh.AuthMethod{
			ssh.Password("secret"),
		},
		HostKeyCallback: ssh.InsecureIgnoreHostKey(),
	}
	client, _ := ssh.Dial("tcp", "host:22", config)
	session, _ := client.NewSession()
	defer session.Close()

	out, _ := session.CombinedOutput("ls -la")
	fmt.Println(string(out))
}
```

## Interview Questions

**Q: What is the difference between `rsync path/` and `rsync path`?**
**A:** The **trailing slash** matters in rsync!
*   `rsync source/ dest`: Copies the *contents* of source into dest.
*   `rsync source dest`: Copies the *directory* source into dest (creating `dest/source`).

**Q: How does `rsync` determine which files to update?**
**A:** By default, it checks file size and modification time. If they differ, it transfers the changes. It does not check content checksums by default (unless `-c` is used) because reading all files is slow.

**Q: What is `StrictHostKeyChecking`?**
**A:** An SSH setting. When you connect to a host for the first time, SSH asks to trust the fingerprint. In automation (scripts/CI), this prompt causes failure. Setting `StrictHostKeyChecking=no` disables the prompt (but increases MITM risk).
