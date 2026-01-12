---
---

# Email Security: SPF, DKIM, DMARC

Email was designed without security, making spoofing ("I am the CEO") trivial. To combat this, three DNS-based mechanisms were added. DevOps engineers often manage these records to ensure transactional emails (password resets, invoices) land in the inbox, not the spam folder.

## Summary

*   **SPF (Sender Policy Framework)**: "Who is allowed to send?" A DNS TXT record listing IP addresses authorized to send mail for a domain.
*   **DKIM (DomainKeys Identified Mail)**: "Did the message change?" A cryptographic signature attached to the email header, verified against a public key in DNS.
*   **DMARC (Domain-based Message Authentication, Reporting, and Conformance)**: "What to do if checks fail?" A policy telling the receiver to Reject, Quarantine, or Allow emails that fail SPF/DKIM.

## Detailed Explanation

### 1. SPF
*   **Record**: `v=spf1 ip4:1.2.3.4 include:_spf.google.com -all`
*   **Mechanism**: The receiver looks up the sender's domain TXT record. If the source IP matches the list, it passes.
*   **Limitation**: Only validates the "Envelope From" (Return-Path), not the "Header From" (what the user sees).

### 2. DKIM
*   **Mechanism**:
    1.  Sender (MTA) calculates a hash of the body+headers.
    2.  Encrypts hash with Private Key -> Adds `DKIM-Signature` header.
    3.  Receiver looks up Public Key in DNS (`selector._domainkey.example.com`).
    4.  Decrypts signature and compares hashes.
*   **Benefit**: Proves the email was not tampered with in transit.

### 3. DMARC
*   **Record**: `v=DMARC1; p=reject; rua=mailto:reports@example.com`
*   **Alignment**: Ensures the "Header From" domain matches the SPF/DKIM domains.
*   **Policy (`p`)**:
    *   `none`: Monitor mode. Log failures but deliver mail.
    *   `quarantine`: Send failures to Spam.
    *   `reject`: Drop failures (Ultimate goal).

---

## Go Implementation Example

Validating these signatures is complex. We generally use a library like `go-msgauth`.

```go
package main

import (
	"fmt"
	"strings"

	"github.com/emersion/go-msgauth/dkim"
)

func main() {
	// Conceptual example of verifying a DKIM signature
	// In reality, you'd feed the raw email stream here
	emailRaw := `DKIM-Signature: v=1; ...
From: boss@company.com
Subject: Wiring Instructions

Please wire $1M to...`

	r := strings.NewReader(emailRaw)
	
	// Verify parses the message and checks the signature against DNS
	verifications, err := dkim.Verify(r)
	if err != nil {
		fmt.Printf("Verification error: %v\n", err)
		return
	}

	for _, v := range verifications {
		if v.Err == nil {
			fmt.Printf("✅ DKIM Valid for domain: %s\n", v.Domain)
		} else {
			fmt.Printf("❌ DKIM Invalid: %v\n", v.Err)
		}
	}
}
```

## Interview Questions

**Q: If I have SPF and DKIM set up, why do I need DMARC?**
**A:** SPF and DKIM are independent checks. A hacker can still spoof your domain in the "From" address while passing SPF/DKIM using *their own* domain (this works because SPF checks the Envelope, not the Header). **DMARC** links the two, requiring "Alignment" (the From header must match the SPF/DKIM domain) and allows you to enforce a `reject` policy for failures.

**Q: What is the difference between `~all` and `-all` in an SPF record?**
**A:**
*   `-all` (Fail): "No other IPs are allowed." Strict.
*   `~all` (SoftFail): "Other IPs are suspicious but allowed." Used during transitions. Many spam filters treat SoftFail almost the same as Fail, but it prevents legitimate mail from being hard-bounced during setup.

**Q: How does DMARC help with visibility?**
**A:** The `rua` tag in DMARC tells receivers (Google, Yahoo) to send you daily XML reports. These reports show every IP sending mail as your domain. This is crucial for DevOps to discover "Shadow IT" (e.g., Marketing using Mailchimp without authorization) before enforcing a strict Reject policy.
