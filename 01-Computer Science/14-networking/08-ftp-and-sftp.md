---
---

## Summary
**FTP (File Transfer Protocol)** and **SFTP (SSH File Transfer Protocol)** are used for transferring files between a client and a server. Despite their similar names, they are completely different protocols.

## Comparison: FTP vs. SFTP

| Feature | FTP | SFTP |
| :--- | :--- | :--- |
| **Full Name** | File Transfer Protocol | SSH File Transfer Protocol |
| **Protocol** | Standalone (Port 21) | Subsystem of SSH (Port 22) |
| **Security** | Plaintext (Insecure) | Encrypted (Secure) |
| **Channels** | Two (Command + Data) | Single (Multiplexed) |
| **Firewall Friendly** | No (Requires passive/active range) | Yes (Single port) |

### FTP Security Note
Standard FTP is insecure. **FTPS** (FTP over SSL/TLS) adds encryption but retains the complexity of dual channels. **SFTP** is the industry standard for secure DevOps workflows.

## Go Implementation: SFTP Client
Using the library `github.com/pkg/sftp`.

```go
package main

import (
	"fmt"
	"io"
	"log"
	"os"

	"github.com/pkg/sftp"
	"golang.org/x/crypto/ssh"
)

func main() {
	// 1. Establish SSH connection first
	config := &ssh.ClientConfig{
		User:            "user",
		Auth:            []ssh.AuthMethod{ssh.Password("pass")},
		HostKeyCallback: ssh.InsecureIgnoreHostKey(),
	}
	conn, err := ssh.Dial("tcp", "example.com:22", config)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	// 2. Create SFTP client
	client, err := sftp.NewClient(conn)
	if err != nil {
		log.Fatal(err)
	}
	defer client.Close()

	// 3. Open a file
	f, err := client.Create("hello.txt")
	if err != nil {
		log.Fatal(err)
	}
	if _, err := f.Write([]byte("Hello from Go!")); err != nil {
		log.Fatal(err)
	}
	f.Close()

	fmt.Println("File uploaded successfully via SFTP")
}
```

## Interview Questions
- **Q: Why is FTP hard to use with firewalls?**
  - **A:** Because it uses separate channels for commands and data. In "Active Mode", the server initiates a connection back to the client, which is often blocked by client-side firewalls. In "Passive Mode", the client connects to a random high port on the server, requiring a large range of ports to be open.
- **Q: Does SFTP require a separate server from SSH?**
  - **A:** No. SFTP is usually a subsystem of the SSH server. If you have SSH access, you typically have SFTP access unless explicitly restricted.
- **Q: What is the main advantage of SFTP over SCP?**
  - **A:** SFTP is a more robust protocol that allows for file manipulation (resuming transfers, directory listing, deleting files), whereas SCP (Secure Copy) is primarily for simple file copying and is now considered legacy.
