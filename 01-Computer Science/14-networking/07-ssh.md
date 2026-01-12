---
---

## Summary
**SSH (Secure Shell)** is a cryptographic network protocol for operating network services securely over an unsecured network. Its most common use case is remote login and command-line execution. It operates at the Application Layer (Layer 7) and typically uses **Port 22**.

## Protocol Basics
SSH uses a client-server model and provides:
1.  **Authentication**: Verifying the identity of the user (Password, Public Key, 2FA).
2.  **Encryption**: Protecting data from eavesdropping.
3.  **Integrity**: Ensuring data hasn't been tampered with.

### Authentication Methods
- **Password**: Simple but vulnerable to brute-force.
- **Public Key**: Recommended for DevOps. Uses an RSA/ED25519 key pair. The public key is placed on the server (`~/.ssh/authorized_keys`), and the private key stays on the client.
- **Certificate-based**: Used in large-scale environments (e.g., Netflix's BLESS, Uber's SSH-CA) to avoid managing individual keys.

### Port Forwarding (SSH Tunneling)
- **Local Forwarding (`-L`)**: Access a remote service on a local port (e.g., `ssh -L 8080:localhost:80 user@remote`).
- **Remote Forwarding (`-R`)**: Expose a local service to a remote port.
- **Dynamic Forwarding (`-D`)**: Turns SSH into a SOCKS proxy.

## Go Implementation: SSH Client
Using the sub-repository package `golang.org/x/crypto/ssh`.

```go
package main

import (
	"fmt"
	"io"
	"log"
	"os"

	"golang.org/x/crypto/ssh"
)

func main() {
	config := &ssh.ClientConfig{
		User: "username",
		Auth: []ssh.AuthMethod{
			ssh.Password("your-password"),
		},
		// WARNING: In production, use HostKeyCallback to verify server identity!
		HostKeyCallback: ssh.InsecureIgnoreHostKey(),
	}

	// Connect to the SSH server
	client, err := ssh.Dial("tcp", "example.com:22", config)
	if err != nil {
		log.Fatal("Failed to dial: ", err)
	}
	defer client.Close()

	// Create a session
	session, err := client.NewSession()
	if err != nil {
		log.Fatal("Failed to create session: ", err)
	}
	defer session.Close()

	// Run a command
	var b []byte
	b, err = session.CombinedOutput("ls -l")
	if err != nil {
		log.Fatal("Failed to run command: ", err)
	}
	fmt.Print(string(b))
}
```

## Interview Questions
- **Q: How does SSH Public Key authentication work?**
  - **A:** The client sends its public key ID. The server generates a random challenge, encrypts it with the client's public key, and sends it back. The client decrypts it with its private key and sends it back (or a hash of it). If it matches, the client is authenticated.
- **Q: What is the purpose of `ssh-agent`?**
  - **A:** It is a helper program that stores your decrypted private keys in memory so you don't have to type your passphrase every time you use SSH.
- **Q: Why should you use `ssh.InsecureIgnoreHostKey()` only for testing?**
  - **A:** Because it bypasses the verification of the server's identity, making the connection vulnerable to Man-in-the-Middle (MitM) attacks. In production, you should verify the host key against a known set of keys.
