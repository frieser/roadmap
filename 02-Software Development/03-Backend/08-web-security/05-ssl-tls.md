---
---

## Summary
TLS (Transport Layer Security) is the successor to SSL (Secure Sockets Layer). It is the cryptographic protocol used to establish the secure connection that makes HTTPS possible.

## Detailed Explanation
While many people still say "SSL", modern web security actually uses **TLS** (specifically TLS 1.2 or 1.3).

### The TLS Handshake (TLS 1.3)
1. **Client Hello**: Client sends supported cipher suites and a random number.
2. **Server Hello & Certificate**: Server chooses the cipher suite, sends its certificate, and its own random number.
3. **Key Exchange**: Both parties generate a shared secret key (using Diffie-Hellman).
4. **Encrypted Communication**: All subsequent data is encrypted with the shared secret key.

### Why TLS 1.3?
TLS 1.3 (released in 2018) is faster and more secure than 1.2. It removed old, vulnerable encryption algorithms and reduced the handshake from two round-trips to one.

## Go Context
Go's `crypto/tls` package handles the low-level details. By default, Go's `http.ListenAndServeTLS` uses secure defaults.

### Example: Configuring TLS versions in Go
```go
package main

import (
	"crypto/tls"
	"net/http"
)

func main() {
	config := &tls.Config{
		MinVersion: tls.VersionTLS12, // Disallow older TLS versions
	}
	server := &http.Server{
		Addr:      ":443",
		TLSConfig: config,
	}
	server.ListenAndServeTLS("cert.pem", "key.pem")
}
```

## Interview Questions
- **Q: What is the difference between SSL and TLS?**
- **A:** SSL is the predecessor to TLS. SSL 3.0 is deprecated and insecure. All modern secure communication uses TLS 1.2 or 1.3, although the term "SSL" is still commonly used in marketing and general conversation.

- **Q: What is a Cipher Suite?**
- **A:** A cipher suite is a set of algorithms that define how the secure connection will be established. It includes algorithms for key exchange, authentication, bulk encryption, and message integrity.

- **Q: What is a Certificate Authority (CA)?**
- **A:** A CA is a trusted third-party organization (like Let's Encrypt or DigiCert) that verifies the identity of a website and issues digital certificates. Browsers trust certificates signed by these CAs.
