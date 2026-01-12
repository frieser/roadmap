#Linux
---
tags: ['linux', 'roadmap', 'tools']
---

## Summary
Understanding how to identify running services and their associated network ports is a fundamental skill for server administration, security auditing, and troubleshooting. This note covers the use of \`systemctl\` for managing service states and modern tools like \`ss\` (Socket Statistics) alongside the classic \`netstat\` for identifying listening sockets and the processes owning them.

## Detailed Explanation

### 1. Listing Services with \`systemctl\`
On modern Linux distributions using \`systemd\`, \`systemctl\` is the primary tool for managing services. To see what is currently active:

\`\`\`bash
# List all active services
systemctl list-units --type=service --state=running

# Check the status of a specific service
systemctl status nginx

# List all services (including inactive ones)
systemctl list-unit-files --type=service
\`\`\`

### 2. Identifying Listening Ports
When a service runs, it often listens on a network port. Identifying which service is listening on which port is crucial for debugging connectivity issues or detecting unauthorized services.

#### The Modern Standard: \`ss\`
The \`ss\` (socket statistics) command is the modern replacement for \`netstat\`. It is faster and provides more detailed information by fetching data directly from the kernel's networking subsystem.

\`\`\`bash
# Common usage to see listening TCP and UDP ports
sudo ss -tulpn
\`\`\`

**Flags Breakdown:**
- \`-t\`: Display **T**CP sockets.
- \`-u\`: Display **U**DP sockets.
- \`-l\`: Display **L**istening sockets.
- \`-p\`: Show the **P**rocess using the socket (requires sudo).
- \`-n\`: Do not resolve names (show **N**umeric port/IP).

#### The Classic Tool: \`netstat\`
While \`netstat\` is deprecated in many distributions in favor of \`iproute2\` tools, it is still widely used in legacy environments.

\`\`\`bash
# Identify listening ports and their PIDs
sudo netstat -tulpn
\`\`\`

### 3. Finding a Specific Port
If you know the port number and want to find the service, you can pipe the output to \`grep\` or use \`lsof\`.

\`\`\`bash
# Find what is listening on port 80
sudo ss -tulpn | grep :80

# Alternative using lsof (List Open Files)
sudo lsof -i :80
\`\`\`

### 4. Summary Table of Tools

| Tool | Purpose | Status |
| :--- | :--- | :--- |
| \`systemctl\` | Manage and list systemd services | Standard |
| \`ss\` | Modern socket statistics tool | Standard |
| \`netstat\` | Classic network connection tool | Deprecated |
| \`lsof\` | Lists open files (including network sockets) | Useful Utility |

## Interview Questions

**Q: What is the main difference between \`netstat\` and \`ss\`?**
**A:** \`ss\` is faster and more efficient because it retrieves information directly from kernel space (via netlink), whereas \`netstat\` reads from \`/proc/net\`, which can be slow on systems with many connections. \`ss\` is part of the \`iproute2\` package, which is the modern standard replacing \`net-tools\`.

**Q: What do the flags \`-tulpn\` mean in the context of network monitoring?**
**A:** They stand for **T**CP, **U**DP, **L**istening, **P**rocess (PID/Name), and **N**umeric (addresses instead of hostnames). Combining them allows you to see all services listening for incoming connections.

**Q: How can you identify which process is listening on port 443 if you don't have \`ss\` or \`netstat\` installed?**
**A:** You can use \`lsof -i :443\` or check the \`/proc\` filesystem directly if necessary, though \`lsof\` is the most common alternative.

**Q: How do you list only the services that failed to start?**
**A:** Use \`systemctl list-units --type=service --state=failed\`.
