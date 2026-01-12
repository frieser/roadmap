---
---

## Summary
Server security involves protecting the underlying infrastructure (OS, network, and hardware) that runs your application. It is a critical component of the "Defense in Depth" strategy.

## Detailed Explanation
Security doesn't stop at the application layer. If your server is compromised, your application is as well.

### Best Practices
- **Keep Software Updated**: Regularly patch the OS and all installed packages.
- **Principle of Least Privilege**: Run processes (like your Go binary) with a non-root user.
- **Firewall Configuration**: Close all ports except those absolutely necessary (e.g., 80, 443).
- **SSH Hardening**: Disable password-based login; use SSH keys instead. Change the default SSH port.
- **Fail2Ban**: Automatically block IP addresses that show malicious behavior (like too many failed SSH attempts).

## Go Context
When deploying Go apps, use minimal Docker images to reduce the attack surface.

### Example: Non-root Dockerfile for Go
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o main .

FROM alpine:latest
RUN adduser -D myuser
USER myuser
COPY --from=builder /app/main /main
CMD ["/main"]
```

## Interview Questions
- **Q: Why should you never run your application as the `root` user?**
- **A:** If an attacker finds a vulnerability in your application (like a remote code execution bug), they will gain the same permissions as the user running the process. If it's `root`, they have full control over the entire server.

- **Q: What is "Defense in Depth"?**
- **A:** It is a security strategy that uses multiple layers of defense. If one layer (like a firewall) is breached, others (like OS hardening or application authentication) are still in place to protect the system.

- **Q: What is a "Bastion Host"?**
- **A:** A Bastion Host (or Jump Box) is a special-purpose server on a network specifically designed and configured to withstand attacks. It is often the only way for administrators to access servers in a private network via SSH.
