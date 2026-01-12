---
---

# SSH (Secure Shell)

SSH is the cryptographic network protocol for operating network services securely over an unsecured network. It is the primary tool for remote server administration, providing a secure channel in a client-server architecture.

## Summary

SSH operates on TCP port 22. It replaces insecure legacy protocols like Telnet and rlogin. It provides strong authentication (Password, Public Key, Host-Based) and encrypted data communication. Beyond shell access, it is used for **Tunneling** (Port Forwarding), **SFTP** (File Transfer), and **X11 Forwarding**.

## Detailed Explanation

### 1. Authentication Methods
*   **Password**: User sends a password (encrypted). Susceptible to brute-force.
*   **Public Key**: The standard for DevOps.
    *   User generates a pair (`id_rsa`, `id_rsa.pub`).
    *   The Public Key is placed in `~/.ssh/authorized_keys` on the server.
    *   The server encrypts a challenge with the Public Key; the client decrypts it with the Private Key to prove identity.

### 2. Port Forwarding (Tunneling)
*   **Local (-L)**: Forward a local port to a remote server.
    *   `ssh -L 8080:localhost:80 user@server` (Access server's web server via localhost:8080).
*   **Remote (-R)**: Forward a remote port to the local machine.
    *   `ssh -R 9000:localhost:3000 user@server` (Expose local dev server to the remote server).
*   **Dynamic (-D)**: Acts as a SOCKS proxy.

### 3. Config Files
*   **Client**: `~/.ssh/config` (Define aliases, keys, and options for hosts).
*   **Server**: `/etc/ssh/sshd_config` (Disable root login, disable passwords, change ports).

---

## Go Implementation Example

The `golang.org/x/crypto/ssh` package allows you to build SSH clients and servers. This is how tools like **Terraform** or **Ansible** (if written in Go) connect to machines.

```go
package main

import (
	"bytes"
	"fmt"
	"log"
	"os"

	"golang.org/x/crypto/ssh"
)

func main() {
	// 1. Read Private Key
	key, err := os.ReadFile("/home/user/.ssh/id_rsa")
	if err != nil {
		log.Fatalf("unable to read private key: %v", err)
	}

	// 2. Parse Private Key
	signer, err := ssh.ParsePrivateKey(key)
	if err != nil {
		log.Fatalf("unable to parse private key: %v", err)
	}

	// 3. Configure Client
	config := &ssh.ClientConfig{
		User: "admin",
		Auth: []ssh.AuthMethod{
			ssh.PublicKeys(signer),
		},
		// WARNING: In production, use ssh.FixedHostKey(public)
		HostKeyCallback: ssh.InsecureIgnoreHostKey(),
	}

	// 4. Connect to Server
	client, err := ssh.Dial("tcp", "example.com:22", config)
	if err != nil {
		log.Fatal("Failed to dial: ", err)
	}
	defer client.Close()

	// 5. Create Session
	session, err := client.NewSession()
	if err != nil {
		log.Fatal("Failed to create session: ", err)
	}
	defer session.Close()

	// 6. Run Command
	var b bytes.Buffer
	session.Stdout = &b
	if err := session.Run("uptime"); err != nil {
		log.Fatal("Failed to run: " + err.Error())
	}
	fmt.Println(b.String())
}
```

## Interview Questions

**Q: Explain the "The authenticity of host 'xyz' can't be established" warning.**
**A:** This is **TOFU (Trust On First Use)**. SSH doesn't rely on a central CA like HTTPS. When you connect to a server for the first time, it presents its Host Key fingerprint. You must manually verify and accept it. SSH stores this fingerprint in `~/.ssh/known_hosts`. If the key changes in the future (reinstall or Man-in-the-Middle attack), SSH will warn you and block the connection.

**Q: What is the difference between `ssh-agent` and `ssh-add`?**
**A:** `ssh-agent` is a background program that holds your decrypted private keys in memory. `ssh-add` is the command used to load keys into the agent. This allows you to use your keys (even those with passphrases) multiple times without re-typing the passphrase or specifying the key path every time.

**Q: How do you harden an SSH server?**
**A:**
1.  Disable Root Login (`PermitRootLogin no`).
2.  Disable Password Authentication (`PasswordAuthentication no`).
3.  Use Protocol 2 only.
4.  Allow specific users (`AllowUsers deploy`).
5.  Change the default port (optional, security through obscurity).
6.  Use tools like Fail2Ban to block brute-force attempts.
