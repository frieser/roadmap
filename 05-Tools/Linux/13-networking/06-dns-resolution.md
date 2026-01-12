#Networking
#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
**DNS Resolution** is the process by which a Linux system translates human-readable domain names (like `google.com`) into machine-readable IP addresses (like `142.250.190.46`). This mechanism relies on a hierarchical distributed database and local configuration files to determine where to send queries and how to handle results. Understanding the order of resolution and the tools used to diagnose it is crucial for system administration and networking.

## Detailed Explanation

### 1. Local Resolution: `/etc/hosts`
Before querying external DNS servers, a Linux system typically checks its local `hosts` file. This is a simple text file that provides a static mapping between IP addresses and hostnames. It is often used for local development or for overriding DNS for specific servers.

```bash
# View the contents of /etc/hosts
cat /etc/hosts

# Typical entry format:
# 127.0.0.1   localhost
# 192.168.1.10 internal-server.local
```

### 2. DNS Resolver Configuration: `/etc/resolv.conf`
The `/etc/resolv.conf` file is the primary configuration file for the DNS resolver library. It specifies the IP addresses of the nameservers the system should contact.

```bash
# View current resolver configuration
cat /etc/resolv.conf

# Common keywords:
# nameserver: IP of the DNS server to query (e.g., 8.8.8.8)
# search: Search list for host-name lookup (e.g., mycompany.com)
# options: Various options to change resolver behavior (e.g., timeout:1)
```

> **Note:** On modern systems using `systemd-resolved` or `NetworkManager`, this file is often a symlink to a dynamically generated file and should not be edited manually.

### 3. Name Service Switch: `/etc/nsswitch.conf`
The order in which the system looks up hostnames is defined in `/etc/nsswitch.conf`. The `hosts` line determines the priority.

```bash
# Check the lookup order
grep hosts /etc/nsswitch.conf

# Example output: hosts: files dns
# This means it checks /etc/hosts (files) before querying DNS (dns).
```

### 4. Core DNS Record Types
- **A Record**: Maps a hostname to an **IPv4** address.
- **AAAA Record**: Maps a hostname to an **IPv6** address.
- **CNAME (Canonical Name)**: An alias that points one domain name to another (e.g., `www.example.com` -> `example.com`).
- **MX (Mail Exchange)**: Specifies the mail servers responsible for receiving email for the domain.
- **NS (Name Server)**: Identifies the authoritative DNS servers for a zone.

### 5. DNS Query Tools

#### `dig` (Domain Information Groper)
`dig` is the most powerful and flexible tool for querying DNS. It provides detailed output about the entire resolution process.

```bash
# Perform a simple lookup
dig google.com

# Get only the IP address (short output)
dig google.com +short

# Reverse DNS lookup (find hostname from IP)
dig -x 8.8.8.8

# Query a specific DNS server (e.g., Cloudflare)
dig @1.1.1.1 google.com
```

#### `nslookup`
A simpler, older tool for querying DNS servers. While less descriptive than `dig`, it is widely available across platforms.

```bash
# Simple lookup
nslookup example.com

# Start interactive mode
nslookup
> set type=MX
> example.com
```

### 6. Recursive vs. Iterative Queries
- **Recursive Query**: The client asks a DNS resolver to find the IP. The resolver does all the work (contacting Root, TLD, and Authoritative servers) and returns the final answer or an error.
- **Iterative Query**: The client asks a DNS server, which returns the best answer it has (e.g., "I don't know, but ask the .com TLD server"). The client then contacts the next server itself.

## Interview Questions

**Q: If you add an entry to `/etc/hosts` but the system ignores it and goes straight to DNS, what file should you check?**
**A:** You should check `/etc/nsswitch.conf`. Look for the `hosts:` line and ensure that `files` appears before `dns`.

**Q: What is the purpose of a CNAME record, and can it point directly to an IP address?**
**A:** A CNAME record is used to create an alias from one domain name to another. It **cannot** point to an IP address; it must point to another domain name (which must eventually resolve to an A or AAAA record).

**Q: How can you find the authoritative nameservers for a specific domain using `dig`?**
**A:** You can query the NS records for the domain: `dig example.com NS`. The "ANSWER SECTION" will list the authoritative servers.

**Q: What does a "TTL" (Time to Live) value in a DNS record signify?**
**A:** TTL is the amount of time (in seconds) that a DNS record can be cached by a resolver before it must be discarded and fetched again from the authoritative server.

**Q: Explain the difference between an A record and a PTR record.**
**A:** An **A record** performs forward resolution (Hostname -> IPv4), while a **PTR record** (Pointer Record) performs reverse resolution (IP -> Hostname), typically found in reverse lookup zones (`in-addr.arpa`).
