#Linux
---
tags: ['linux', 'roadmap', 'tools']
---

## Summary
`netstat` (network statistics) is a command-line tool used for monitoring network connections (both incoming and outgoing), routing tables, interface statistics, masquerade connections, and multicast memberships. While technically deprecated in favor of the faster `ss` command, it remains one of the most widely used tools for network troubleshooting in Linux environments.

## Detailed Explanation

### 1. Overview
`netstat` is part of the `net-tools` package. It provides a "snapshot" of the networking subsystem. In modern Linux distributions, it is often superseded by `ss` (Socket Statistics) because `ss` retrieves information directly from kernel space (using netlink), making it significantly faster and more accurate on systems with thousands of active connections.

### 2. Common Flags and Usage
The most common invocation for troubleshooting is `netstat -tulpn`.

| Flag | Description |
| :--- | :--- |
| **-t** | Display **TCP** connections. |
| **-u** | Display **UDP** connections. |
| **-l** | Show only **Listening** sockets (omits established connections). |
| **-p** | Show the **PID** and name of the program owning the socket (requires `sudo`). |
| **-n** | **Numeric** output: shows IP addresses and port numbers instead of resolving hostnames/services. |
| **-a** | Show **All** sockets (both listening and non-listening). |
| **-r** | Display the kernel **Routing** table (similar to `route -n`). |
| **-i** | Display a table of all network **Interfaces**. |

### 3. Understanding Connection States
When viewing active TCP connections, you will see various states:
*   **LISTEN**: The socket is waiting for an incoming connection.
*   **ESTABLISHED**: The socket has an active, successful connection.
*   **CLOSE_WAIT**: The remote end has shut down, and the local system is waiting for the application to close the socket.
*   **TIME_WAIT**: The socket is closed, but the system is keeping it around to ensure any delayed packets are handled correctly.
*   **SYN_SENT**: The local system is actively attempting to establish a connection.

### 4. Netstat vs. ss
| Feature | netstat | ss |
| :--- | :--- | :--- |
| **Package** | `net-tools` (Deprecated) | `iproute2` (Modern) |
| **Performance** | Slower (parses `/proc/net/*`) | Faster (queries kernel via netlink) |
| **Accuracy** | May miss rapid state changes | More accurate for high-load servers |
| **Syntax** | `netstat -tulpn` | `ss -tulpn` (Very similar) |

### 5. Bash Examples

**List all listening ports and their processes:**
```bash
sudo netstat -tulpn
```

**Check if a specific port (e.g., 80) is being used:**
```bash
sudo netstat -tulpn | grep :80
```

**Display the routing table:**
```bash
netstat -rn
```

**Display network interface statistics:**
```bash
netstat -ie
```

## Interview Questions

1. **What is the difference between `netstat` and `ss`?**
   *   `ss` is part of the `iproute2` package and is the modern replacement for `netstat`. `ss` is faster because it gets information directly from the kernel using the netlink protocol, whereas `netstat` reads and parses several files in `/proc/net/`.

2. **Why do you need `sudo` when running `netstat -p`?**
   *   The `-p` flag attempts to identify the Process ID (PID) and program name associated with a socket. For security reasons, a regular user cannot see process information for sockets owned by other users or the system; `root` privileges are required to map all sockets to their respective processes.

3. **How would you find out which process is listening on port 443?**
   *   You can run `sudo netstat -tulpn | grep :443`. This will filter the output to show only the line containing the port, including the PID and process name (e.g., `nginx` or `apache`).

4. **What does the `TIME_WAIT` state indicate, and can it be a problem?**
   *   `TIME_WAIT` means the local end has closed the connection, but the kernel keeps the socket record to handle any stray packets still "in flight" on the network. While normal, a very high number of `TIME_WAIT` sockets can eventually exhaust the available local port range (ephemeral ports) on extremely busy servers.

5. **How can you see the routing table using `netstat`?**
   *   Use `netstat -r`. Adding the `-n` flag (`netstat -rn`) is usually preferred to prevent the command from hanging while it tries to perform DNS lookups for every IP in the table.
