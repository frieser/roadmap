---
---

## Summary
**DNS (Domain Name System)** is the distributed, hierarchical naming system for computers and services connected to the Internet. Often called the "phonebook of the Internet," its primary role is translating human-friendly domain names (e.g., \`google.com\`) into machine-readable IP addresses (e.g., \`142.250.190.46\`). It operates at the Application Layer (Layer 7) of the OSI model.

## The Distributed Hierarchy
DNS is not a single database but a tree-like structure distributed across millions of servers:

1.  **Root Name Servers (.)**: The top of the hierarchy. There are 13 logical root servers (represented by hundreds of physical locations via Anycast). They direct queries to the appropriate TLD nameserver.
2.  **Top-Level Domain (TLD) Servers**: Responsible for domains ending in specific extensions like \`.com\`, \`.org\`, or country codes like \`.es\`.
3.  **Authoritative Name Servers**: The final stop. These servers hold the actual records for a specific domain (e.g., the records for \`example.com\`).

## Resolution Process: Recursive vs. Iterative

### 1. Recursive Query
The client (Resolver) asks a **Recursive Resolver** (typically provided by your ISP or services like 8.8.8.8) to find the IP. The resolver is obligated to return either the answer or an error. It does all the heavy lifting of querying other servers.

### 2. Iterative Query
The Recursive Resolver performs **Iterative Queries** to the hierarchy. It asks a server, which responds with a "referral" (e.g., "I don't know, but ask the .com TLD server at this IP"). The resolver then asks the next server itself.

\`\`\`mermaid
sequenceDiagram
    participant Client
    participant Resolver
    participant Root
    participant TLD
    participant Auth

    Note over Client, Auth: Recursive Query
    Client->>Resolver: Where is google.com?
    
    Note over Resolver, Auth: Iterative Queries
    Resolver->>Root: Where is .com?
    Root-->>Resolver: Referral to .com TLD (IP)
    Resolver->>TLD: Where is google.com?
    TLD-->>Resolver: Referral to Google Auth NS (IP)
    Resolver->>Auth: What is google.com's IP?
    Auth-->>Resolver: 142.250.190.46
    
    Resolver-->>Client: 142.250.190.46
\`\`\`

## Technical Details

### Ports and Protocols
*   **Port 53**: The standard port for DNS.
*   **UDP (The default)**: Used for standard queries because of low overhead and speed.
    *   **The 512-byte Limit**: Historically, DNS over UDP was limited to 512 bytes to avoid IP fragmentation (IPv4 guarantees 576 bytes reassembly; minus headers, 512 remains).
    *   **EDNS0**: Modern DNS uses Extension Mechanisms for DNS (EDNS0) to allow larger UDP packets (often up to 4096 bytes).
*   **TCP Fallback**: If a DNS response is too large for UDP (indicated by the **TC - Truncation bit**), the client retries the query over TCP.
*   **TCP for Zone Transfers**: Operations like AXFR (full zone transfer) always use TCP due to data size and reliability requirements.

### Core Record Types
| Type | Name | Purpose |
| :--- | :--- | :--- |
| **A** | Address | Maps a hostname to an **IPv4** address. |
| **AAAA** | IPv6 Address | Maps a hostname to an **IPv6** address. |
| **CNAME** | Canonical Name | An alias. Points one domain to another (e.g., \`www.example.com\` -> \`example.com\`). |
| **MX** | Mail Exchange | Directs email to a mail server. |
| **NS** | Name Server | Delegates a DNS zone to use specific authoritative servers. |
| **PTR** | Pointer | Reverse DNS: Maps an IP address to a hostname. |
| **TXT** | Text | Stores arbitrary text (used for SPF, DKIM, and site verification). |

## Go Implementation
Using the standard \`net\` package is common, but for advanced needs, \`github.com/miekg/dns\` is the industry standard.

### Using \`net.Resolver\` (Standard)
\`\`\`go
package main

import (
	"context"
	"fmt"
	"net"
	"time"
)

func main() {
	r := &net.Resolver{
		PreferGo: true,
		Dial: func(ctx context.Context, network, address string) (net.Conn, error) {
			d := net.Dialer{Timeout: 10 * time.Second}
			return d.DialContext(ctx, network, "8.8.8.8:53") // Force specific resolver
		},
	}
	ips, _ := r.LookupHost(context.Background(), "google.com")
	fmt.Println("IPs:", ips)
}
\`\`\`

### Using \`miekg/dns\` (Advanced)
\`\`\`go
package main

import (
	"fmt"
	"github.com/miekg/dns"
)

func main() {
	c := new(dns.Client)
	m := new(dns.Msg)
	m.SetQuestion(dns.Fqdn("google.com"), dns.TypeA)
	
	// Exchange sends the message and waits for a response
	in, _, err := c.Exchange(m, "8.8.8.8:53")
	if err != nil {
		panic(err)
	}

	for _, answer := range in.Answer {
		if a, ok := answer.(*dns.A); ok {
			fmt.Printf("Record found: %s\n", a.A)
		}
	}
}
\`\`\`

## Interview Questions
**Q: Why does DNS prefer UDP over TCP, and when does it switch?**
**A:** UDP is faster and stateless, reducing server load for billions of queries. It switches to TCP if the response exceeds 512 bytes (without EDNS0) or the EDNS0 buffer size, signaled by the Truncation (TC) bit. Zone transfers also use TCP for reliability.

**Q: What is DNS TTL (Time To Live)?**
**A:** TTL is a value in a DNS record that tells resolvers how long (in seconds) they should cache the record before asking the authoritative server again. High TTL reduces load but slows down updates; low TTL allows fast changes but increases latency.

**Q: What is a Root Nameserver, and how many are there?**
**A:** Root nameservers are the first step in resolving a domain. They don't know the IP of \`google.com\`, but they know who handles \`.com\`. There are 13 logical addresses (A-M), but they are implemented as hundreds of servers globally using Anycast.

**Q: Explain DNS Rebinding.**
**A:** A security attack where a malicious site uses short TTLs to switch its domain's IP from a public one to a private internal IP (like \`127.0.0.1\`) after the browser has already authorized the domain, potentially bypassing Same-Origin Policy.

**Q: What is the purpose of an NS record?**
**A:** It identifies the authoritative servers for a domain. When you buy a domain at a registrar, you set NS records to point to your DNS provider (e.g., Route53, Cloudflare).
