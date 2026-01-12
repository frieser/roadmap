---
---

# FTP vs. SFTP

File transfer is a common requirement in infrastructure, from uploading backups to deploying artifacts. While they sound similar, FTP and SFTP are completely different protocols with vastly different security profiles.

## Summary

*   **FTP (File Transfer Protocol)**: An old, insecure protocol (1971) that sends passwords and data in clear text. It uses two separate channels (Command and Data) which makes it firewall-unfriendly.
*   **SFTP (SSH File Transfer Protocol)**: An extension of the SSH protocol. It uses a single encrypted channel (port 22). It is secure, firewall-friendly, and the standard for modern DevOps.

## Detailed Explanation

### FTP (The Old Way)
*   **Ports**: 21 (Command) and 20 (Data).
*   **Modes**:
    *   **Active**: Server connects back to Client (Nightmare for client firewalls).
    *   **Passive**: Client connects to Server on a random high port (Easier for firewalls).
*   **FTPS**: FTP over SSL. Adds encryption but keeps the complex multi-port architecture.

### SFTP (The DevOps Way)
*   **Port**: 22 (Same as SSH).
*   **Mechanism**: It is a subsystem of SSH. If you can SSH into a box, you can usually SFTP.
*   **Features**: Resume interrupted transfers, directory listing, remote file removal.

---

## Go Implementation Example

Using the `github.com/pkg/sftp` library, we can interact with an SFTP server. This requires an underlying SSH connection first.

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/pkg/sftp"
	"golang.org/x/crypto/ssh"
)

func main() {
	// 1. Setup SSH Configuration (Same as SSH example)
	config := &ssh.ClientConfig{
		User: "admin",
		Auth: []ssh.AuthMethod{
			ssh.Password("secret"),
		},
		HostKeyCallback: ssh.InsecureIgnoreHostKey(),
	}

	// 2. Connect via SSH
	conn, err := ssh.Dial("tcp", "example.com:22", config)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	// 3. Initialize SFTP Client
	client, err := sftp.NewClient(conn)
	if err != nil {
		log.Fatal(err)
	}
	defer client.Close()

	// 4. Create Remote File
	f, err := client.Create("/tmp/hello.txt")
	if err != nil {
		log.Fatal(err)
	}
	
	// 5. Write Data
	if _, err := f.Write([]byte("Hello SFTP World!")); err != nil {
		log.Fatal(err)
	}
	f.Close()

	// 6. List Directory
	files, _ := client.ReadDir("/tmp")
	for _, file := range files {
		fmt.Println(file.Name())
	}
}
```

## Interview Questions

**Q: Why is FTP considered "firewall unfriendly"?**
**A:** FTP uses dynamic secondary ports for data transfer. In Passive mode, the server tells the client "Connect to me on port 50000 for data." The firewall must allow this random high port. In Active mode, the server tries to connect back to the client, which is almost always blocked by client-side NAT/Firewalls.

**Q: Is SFTP related to FTPS?**
**A:** No. **FTPS** is the old FTP protocol wrapped in SSL/TLS. It still has the multi-port issues. **SFTP** is a completely different protocol built from the ground up to run over SSH packets. They share no code or logic.

**Q: How do you restrict a user to only SFTP (no shell access)?**
**A:** In `/etc/ssh/sshd_config`, use `Match User` or `Match Group`. Set `ForceCommand internal-sftp` and `ChrootDirectory /home/%u`. This locks the user into their folder and prevents them from running shell commands (`ssh user@host` will fail, but `sftp user@host` will work).
