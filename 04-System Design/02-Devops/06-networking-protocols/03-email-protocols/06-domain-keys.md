---
---

## Summary
DKIM (DomainKeys Identified Mail) is an email authentication technique that allows the receiver to check that an email was indeed authorized by the owner of that domain. It uses digital signatures (public-key cryptography) to verify that the email was not altered in transit.

## Detailed Explanation

### How it Works
1.  **Signing (Sending Server)**:
    *   The server calculates a hash of the email headers and body.
    *   It encrypts this hash using its **Private Key**.
    *   It attaches this signature as a header: `DKIM-Signature`.
2.  **Publishing**:
    *   The domain owner publishes the **Public Key** in a DNS TXT record.
3.  **Verifying (Receiving Server)**:
    *   The receiver sees the `DKIM-Signature` header.
    *   It retrieves the Public Key from the sender's DNS.
    *   It decrypts the signature to get the original hash.
    *   It calculates its own hash of the received email.
    *   If the hashes match, the email is authentic and unaltered.

### The Selector
DKIM uses a "selector" to allow multiple keys for the same domain (e.g., `google._domainkey.example.com`). The header specifies which selector was used: `s=google`.

## Go-Specific Context/Examples

Go libraries like `github.com/emersion/go-msgauth` can be used to sign or verify emails.

### Example: Verifying a DKIM Record via DNS

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	// Format: selector._domainkey.domain
	selector := "20230601"
	domain := "gmail.com"
	query := fmt.Sprintf("%s._domainkey.%s", selector, domain)

	txtRecords, err := net.LookupTXT(query)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}

	for _, record := range txtRecords {
		fmt.Printf("DKIM Public Key for %s:\n%s\n", domain, record)
	}
}
```

## Interview Questions

**Q: Does DKIM encrypt the email content?**
**A:** No. DKIM only provides an *integrity check* and *authentication*. The email content is still plain text (unless S/MIME or PGP is used). DKIM ensures the content wasn't modified and the sender is who they say they are.

**Q: How does DKIM solve the SPF forwarding problem?**
**A:** Because the DKIM signature is part of the email header, it survives forwarding. When Server B forwards the email to Server C, the signature remains valid as long as the body and signed headers are not modified.

**Q: What is DMARC's relationship to SPF and DKIM?**
**A:** DMARC (Domain-based Message Authentication, Reporting, and Conformance) relies on SPF and DKIM. It tells the receiver what to do if SPF or DKIM fails (e.g., "Reject it" or "Quarantine it"). It unifies the results of both protocols.
