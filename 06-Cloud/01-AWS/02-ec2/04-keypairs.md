#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ec2']
---

## Summary
**EC2 Key Pairs** are the primary mechanism for secure authentication when connecting to Amazon EC2 instances. Based on **Public Key Cryptography**, they eliminate the need for traditional passwords, providing a more secure way to log in via SSH (Linux) or retrieve Administrator passwords (Windows). A key pair consists of a **Public Key** stored by AWS on the instance and a **Private Key** kept securely by the user.

## Detailed Explanation

### 1. Public-Key Cryptography Architecture
AWS uses asymmetric encryption to secure your instances:
- **Public Key**: When you launch an instance and specify a key pair, AWS places the public key in the instance's metadata and injects it into the `~/.ssh/authorized_keys` file (for Linux) or uses it to encrypt the initial password (for Windows).
- **Private Key**: You download this file once (usually `.pem` or `.ppk`). It is your digital signature. **AWS does not store the private key**; if you lose it, you cannot download it again.

### 2. Key Algorithms
AWS supports two types of key pair algorithms:
- **RSA**: The industry standard, compatible with almost all operating systems and versions.
- **ED25519**: A modern, high-performance elliptic curve algorithm. It is more secure and faster than RSA but requires modern AMIs (e.g., Amazon Linux 2023, Ubuntu 20.04+, RHEL 8+).

### 3. File Formats: PEM vs PPK
The format of your private key depends on your SSH client:
- **.pem (Privacy Enhanced Mail)**: Used by OpenSSH, macOS Terminal, Linux Terminal, and Windows 10/11 native OpenSSH client.
- **.ppk (PuTTY Private Key)**: Proprietary format used by the **PuTTY** client on Windows. 
- **Conversion**: You can convert `.pem` to `.ppk` (and vice versa) using **PuTTYgen**.

### 4. Creating vs. Importing Key Pairs
- **AWS Generated**: You ask AWS to create the key. AWS generates the pair, stores the public key, and hands you the private key for download.
- **User Imported**: You generate the key pair on your local machine (e.g., `ssh-keygen -t rsa -b 4096`). You then upload the **public key** to AWS. This is considered a best practice in security-conscious organizations to ensure the private key never leaves the local environment.

### 5. Platform Specifics
- **Linux/Unix**: You connect via SSH. The default user depends on the AMI:
  - `ec2-user` (Amazon Linux)
  - `ubuntu` (Ubuntu)
  - `admin` (Debian)
  - `root` or `fedora` (Fedora/RHEL)
- **Windows**: You use the private key to decrypt the randomly generated **Administrator password** via the AWS Console/API. Once decrypted, you use that password with Remote Desktop Protocol (RDP).

### 6. Security & Best Practices
- **File Permissions**: On Linux/macOS, you must set permissions to `400` (`chmod 400 mykey.pem`). If the file is world-readable, SSH will reject it for security reasons.
- **Key Rotation**: Regularly rotate keys by adding new public keys to `authorized_keys` and removing old ones.
- **Least Privilege**: Avoid sharing one key pair across multiple users. Instead, use **IAM Roles** with **EC2 Instance Connect** or **AWS Systems Manager Session Manager** for better auditability and security without long-lived keys.
- **Limits**: There is a limit of **5,000 key pairs per region**.

## Interview Questions

**1. What happens if you lose the private key for an EC2 instance?**
AWS does not store the private key, so it cannot be recovered. However, you can regain access by:
1. Using **EC2 Instance Connect** or **SSM Session Manager** if they were previously configured.
2. Using a "Rescue Instance": Stop the instance, detach the root volume, attach it to another instance as a data volume, manually edit the `authorized_keys` file, then move the volume back.
3. Using an automation document in Systems Manager (e.g., `AWSSupport-ResetAccess`).

**2. Why do you receive a "Permissions are too open" error when using a PEM file?**
SSH clients require that private keys are kept strictly private. If the file permissions allow other users on your local system to read it (e.g., `644`), the client will refuse to use it. On Linux/macOS, use `chmod 400 <file>.pem`. On Windows, you must edit the file security properties to remove all users except yourself.

**3. Can you associate a new key pair with a running EC2 instance?**
You cannot change the key pair associated with the instance metadata after launch. However, you can manually add any number of public keys to the `~/.ssh/authorized_keys` file inside the OS. The "Key Pair Name" shown in the AWS Console will still reflect the original key used at launch.

**4. How does a key pair work for a Windows instance compared to a Linux instance?**
For Linux, the key pair is used for direct SSH authentication. For Windows, the key pair is used to encrypt the initial Administrator password. The user must provide the private key to the AWS Console/API to decrypt and retrieve the password, which is then used for RDP login.

**5. When would you choose ED25519 over RSA?**
You would choose ED25519 for better security and performance (shorter keys, faster computation) if your target operating system supports it (Modern Linux distros). You would stick with RSA for maximum compatibility, especially with older legacy AMIs or specific corporate tools that don't yet support elliptic curve keys.
