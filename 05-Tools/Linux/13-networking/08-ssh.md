---
tags: ['linux', 'roadmap']
---

# SSH (Secure Shell)

## Summary
SSH (Secure Shell) is a cryptographic network protocol used for secure remote login and other secure network services over an insecure network. It provides a secure channel over an unencrypted network in a client-server architecture, connecting an SSH client application with an SSH server. SSH replaced insecure protocols like Telnet and rlogin by providing strong encryption, public-key authentication, and data integrity.

## Detailed Explanation

### 1. Key-Based Authentication
Instead of relying on passwords, SSH uses asymmetric cryptography (public/private keys) to authenticate users.

*   **Private Key (`id_rsa`, `id_ed25519`)**: Stored on the client machine. It must be kept secret and should be protected with a passphrase.
*   **Public Key (`id_rsa.pub`, `id_ed25519.pub`)**: Shared with any server you want to access.

#### Generating an SSH Key
The modern recommendation is to use the **Ed25519** algorithm for better security and performance compared to RSA.

```bash
# Generate a new Ed25519 key with a comment (usually email)
ssh-keygen -t ed25519 -C "user@example.com"
```

### 2. The Server Side: `authorized_keys`
For a server to allow you to log in via a public key, your public key must be listed in the user's `~/.ssh/authorized_keys` file on that server.

```bash
# Copying your public key to a remote server automatically
ssh-copy-id user@remote-host
```

### 3. The Client Side: `known_hosts`
When you connect to a server for the first time, SSH shows you the server's public key fingerprint. If you accept it, the fingerprint is saved in `~/.ssh/known_hosts`. This prevents **Man-in-the-Middle (MITM)** attacks by ensuring you are connecting to the same server next time.

### 4. SSH Agent (`ssh-agent`)
If your private key is protected by a passphrase, you would normally have to type it every time you use the key. The `ssh-agent` stores your decrypted keys in memory for the duration of your session.

```bash
# Start the agent in the background
eval "$(ssh-agent -s)"

# Add your private key to the agent
ssh-add ~/.ssh/id_ed25519
```

### 5. SSH Client Configuration (`~/.ssh/config`)
The config file allows you to create aliases for long SSH commands, specify ports, users, and even proxy jumps.

**Example `~/.ssh/config` file:**
```text
Host dev-server
    HostName 192.168.1.50
    User devuser
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

Host prod-server
    HostName 10.0.0.100
    User admin
    ProxyJump dev-server
```
Now, instead of `ssh -p 2222 devuser@192.168.1.50`, you can just run:
```bash
ssh dev-server
```

### 6. Common SSH Commands

```bash
# Securely copy a file to a remote server
scp local_file.txt user@remote-host:/path/to/destination/

# Securely copy a directory from a remote server
scp -r user@remote-host:/path/to/remote_dir/ ./local_dir/

# Local Port Forwarding (Access remote database on local port 8080)
ssh -L 8080:localhost:5432 user@remote-host
```

## Interview Questions

**Q: What is the difference between `authorized_keys` and `known_hosts`?**
**A:** `authorized_keys` is on the **server** and contains the public keys of clients allowed to log in. `known_hosts` is on the **client** and contains the public keys (fingerprints) of servers the client has previously connected to, used to verify the server's identity.

**Q: Why is Ed25519 preferred over RSA for SSH keys?**
**A:** Ed25519 is faster, more secure (smaller keys provide better security than much larger RSA keys), and generates shorter signatures. RSA with less than 2048 bits is now considered insecure.

**Q: How do you fix a "Host key verification failed" error?**
**A:** This happens when the server's host key doesn't match the one stored in `known_hosts` (often after a server re-install). You can remove the old key using `ssh-keygen -R <hostname>` or manually editing `~/.ssh/known_hosts`.

**Q: What is a ProxyJump and why is it useful?**
**A:** ProxyJump (or the `JumpHost` pattern) allows you to connect to a server in a private network by "jumping" through a gateway server (bastion host) that is accessible from the public internet, without needing to manually SSH twice or use complex tunnels.

**Q: What does the `ssh-agent` do?**
**A:** It is a helper program that keeps track of user's identity keys and their passphrases. It allows the user to log in to different servers without having to re-enter their passphrase for every connection.
