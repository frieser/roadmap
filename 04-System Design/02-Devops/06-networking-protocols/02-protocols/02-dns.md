---
---

# DNS (Domain Name System)

DNS is the "phonebook" of the internet, translating human-readable domain names (like `google.com`) into machine-readable IP addresses (like `142.250.190.46`). For DevOps, DNS is critical for service discovery, load balancing, and routing.

## Summary

DNS acts as a hierarchical, distributed database. When a user queries a domain, the request travels through a chain of resolvers: from the local **Stub Resolver** -> **Recursive Resolver** (ISP/Public) -> **Root Server** -> **TLD Server** -> **Authoritative Nameserver**.

## Detailed Explanation

### 1. Common Record Types
*   **A**: Maps a hostname to an **IPv4** address.
*   **AAAA**: Maps a hostname to an **IPv6** address.
*   **CNAME (Canonical Name)**: Maps an alias to another hostname. (e.g., `www.example.com` -> `example.com`). *Note: Cannot coexist with other records at the root.*
*   **MX (Mail Exchange)**: Specifies mail servers for the domain.
*   **TXT**: Arbitrary text. Used heavily for verification (SPF, DKIM, Google Site Verification).
*   **NS (Name Server)**: Delegates a subdomain to a specific set of name servers.

### 2. The Resolution Process
1.  **Browser Cache**: Checks if the IP is cached locally.
2.  **OS/Recursive Resolver**: The OS queries the configured DNS server (e.g., 8.8.8.8).
3.  **Root Hints**: If unknown, the recursive resolver asks a Root Server (e.g., `a.root-servers.net`).
4.  **TLD Server**: The Root refers to the `.com` TLD server.
5.  **Authoritative Server**: The TLD server refers to the actual nameserver hosting the domain (e.g., AWS Route53).
6.  **Answer**: The Authoritative server returns the IP.

### 3. TTL (Time To Live)
The duration (in seconds) that a DNS record is cached by resolvers.
*   **High TTL (24h)**: Reduces traffic, but changes take longer to propagate.
*   **Low TTL (60s)**: Good for failover/migrations, but increases load.

---

## Go Implementation Example

Go's `net` package provides a robust DNS resolver.

```go
package main

import (
	"context"
	"fmt"
	"net"
	"time"
)

func main() {
	// Custom Resolver (optional, defaults to system)
	r := &net.Resolver{
		PreferGo: true,
		Dial: func(ctx context.Context, network, address string) (net.Conn, error) {
			d := net.Dialer{
				Timeout: time.Millisecond * time.Duration(10000),
			}
			// Force using Google DNS for this example
			return d.DialContext(ctx, network, "8.8.8.8:53")
		},
	}

	domain := "google.com"
	ctx := context.Background()

	// 1. Lookup IP Addresses (A / AAAA)
	ips, err := r.LookupIP(ctx, "ip", domain)
	if err != nil {
		fmt.Printf("Could not get IPs: %v\n", err)
	} else {
		fmt.Printf("IPs for %s:\n", domain)
		for _, ip := range ips {
			fmt.Printf("- %s\n", ip.String())
		}
	}

	// 2. Lookup TXT Records (often used for verification)
	txts, err := r.LookupTXT(ctx, domain)
	if err != nil {
		fmt.Printf("Could not get TXT records: %v\n", err)
	} else {
		fmt.Printf("\nTXT Records:\n")
		for _, t := range txts {
			fmt.Printf("- %s\n", t)
		}
	}
	
	// 3. Lookup MX Records
	mxs, err := r.LookupMX(ctx, domain)
	if err != nil {
		fmt.Printf("Could not get MX records: %v\n", err)
	} else {
		fmt.Printf("\nMX Records:\n")
		for _, mx := range mxs {
			fmt.Printf("- %s %d\n", mx.Host, mx.Pref)
		}
	}
}
```

## Interview Questions

**Q: What is the difference between an Iterative and a Recursive DNS query?**
**A:**
*   **Recursive**: The client (you) asks the server (ISP) "Give me the IP for google.com" and expects the final answer. The server does all the legwork.
*   **Iterative**: The server replies "I don't know, but here is the address of the server that might know (e.g., the .com server)." The client must then query that next server.

**Q: Why can't you have a CNAME at the root domain (example.com)?**
**A:** The DNS specification (RFC 1034) states that if a CNAME record exists for a name, no other data can exist for that name. Since the root domain *must* have SOA and NS records, it cannot also have a CNAME. This is why providers like AWS Route53 introduced "Alias" records to bypass this limitation.

**Q: How does a "Split-Horizon" DNS work?**
**A:** It is a configuration where the DNS server provides different answers depending on the source IP of the request. For example, internal employees querying `jira.company.com` get a private IP (`10.x.x.x`), while users on the public internet get a public IP (`54.x.x.x`) or nothing at all.
