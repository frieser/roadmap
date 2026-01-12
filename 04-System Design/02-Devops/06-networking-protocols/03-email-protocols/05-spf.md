---
---

## Summary
SPF (Sender Policy Framework) is an email authentication method designed to detect forged sender addresses (spoofing) during the delivery of the email. It allows the owner of a domain to specify which mail servers are authorized to send email on behalf of that domain via a DNS record.

## Detailed Explanation

### How it Works
1.  **Publisher**: The domain owner publishes a DNS TXT record starting with `v=spf1`.
    *   Example: `v=spf1 ip4:192.168.0.1 include:_spf.google.com -all`
2.  **Receiver**: When a mail server receives an email claiming to be from `user@example.com`:
    *   It extracts the domain (`example.com`).
    *   It queries DNS for the SPF record of `example.com`.
    *   It checks if the connecting IP address is listed in the record.
3.  **Result**:
    *   **Pass**: IP is authorized.
    *   **Fail/SoftFail**: IP is not authorized (Email might be marked as spam or rejected).

### Record Syntax
*   `+`: Pass (Default).
*   `-`: Fail (Hard failure, reject).
*   `~`: SoftFail (Accept but mark, often used during transitions).
*   `include:`: Trust the SPF record of another domain (e.g., your email provider).

## Go-Specific Context/Examples

You can use Go's `net` package to lookup SPF records.

### Example: Looking up SPF Records

```go
package main

import (
	"fmt"
	"net"
	"strings"
)

func main() {
	domain := "gmail.com"
	
	txtRecords, err := net.LookupTXT(domain)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}

	for _, record := range txtRecords {
		if strings.HasPrefix(record, "v=spf1") {
			fmt.Printf("Found SPF Record for %s:\n%s\n", domain, record)
		}
	}
}
```

## Interview Questions

**Q: What is the limitation of SPF regarding email forwarding?**
**A:** SPF breaks when emails are forwarded. If Server A sends to Server B (Passes SPF), and Server B forwards to Server C, Server C sees the connection coming from Server B, but the "From" address is still the original domain. Since Server B is not in the original domain's SPF record, the check fails. (DKIM solves this).

**Q: What does `-all` vs `~all` mean at the end of an SPF record?**
**A:** `-all` (Hard Fail) means "Reject any mail not from these IPs". `~all` (Soft Fail) means "Accept but mark as suspicious/spam". Many domains use `~all` to avoid legitimate email loss due to misconfiguration or forwarding issues.

**Q: Can you have multiple SPF records for a domain?**
**A:** No. A domain must have exactly one SPF TXT record. If you have multiple services (Google, Mailchimp, Outlook), you must combine them into a single `include` statement. Multiple records cause validation to fail permanently (PermError).
