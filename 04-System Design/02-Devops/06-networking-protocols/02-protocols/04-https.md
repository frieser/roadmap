---
---

# HTTPS (Hypertext Transfer Protocol Secure)

HTTPS is the secure version of HTTP. It uses SSL/TLS to encrypt communications between a client and server, ensuring confidentiality, integrity, and authenticity. In modern DevOps, HTTPS is mandatory; search engines penalize non-HTTPS sites, and new browser features (like Service Workers) require it.

## Summary

HTTPS is simply HTTP layered over **TLS (Transport Layer Security)**. It operates on TCP port 443 by default. The protocol ensures that data cannot be intercepted (Confidentiality), modified (Integrity), or spoofed (Authenticity) by attackers.

## Detailed Explanation

### 1. Encryption
HTTPS uses **Asymmetric Encryption** (Public/Private Key) for the initial handshake to exchange a shared secret, and then switches to **Symmetric Encryption** (AES, ChaCha20) for the actual data transfer because it is much faster.

### 2. Certificates (PKI)
*   **Certificate Authority (CA)**: A trusted entity (like Let's Encrypt, DigiCert) that issues certificates.
*   **Chain of Trust**: The browser trusts the Root CA. The Root CA trusts the Intermediate CA. The Intermediate CA signs your domain's certificate.
*   **Validation**:
    *   **DV (Domain Validation)**: Checks you own the domain (DNS/HTTP challenge).
    *   **OV/EV (Organization/Extended Validation)**: Verifies the legal entity behind the site.

### 3. HTTP Strict Transport Security (HSTS)
A security header (`Strict-Transport-Security`) that tells browsers to *only* access the site via HTTPS for a specified period, preventing protocol downgrade attacks (SSL Stripping).

---

## Go Implementation Example

Go makes running an HTTPS server trivial with `ListenAndServeTLS`. For a client, you might need to configure the `TLSClientConfig` if dealing with self-signed certificates (common in internal DevOps environments).

```go
package main

import (
	"crypto/tls"
	"fmt"
	"log"
	"net/http"
)

func main() {
	// --- HTTPS Server ---
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Secure Hello! Protocol: %s", r.Proto)
	})

	// To run this, you need a cert.pem and key.pem
	// Generate self-signed: `openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem`
	go func() {
		fmt.Println("HTTPS Server listening on :8443")
		// ListenAndServeTLS automatically handles the handshake
		err := http.ListenAndServeTLS(":8443", "cert.pem", "key.pem", nil)
		if err != nil {
			log.Fatal(err)
		}
	}()

	// --- HTTPS Client (with insecure skip verify for self-signed) ---
	// In production, NEVER use InsecureSkipVerify: true
	tr := &http.Transport{
		TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
	}
	client := &http.Client{Transport: tr}

	// Wait for server to start... (use proper synchronization in real code)
	// Make request
	resp, err := client.Get("https://localhost:8443")
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	fmt.Printf("Client connected via: %s\n", resp.TLS.Version) // e.g., 0x0304 for TLS 1.3
}
```

## Interview Questions

**Q: How does HTTPS prevent "Man-in-the-Middle" (MitM) attacks?**
**A:** It uses the **Chain of Trust**. The server presents a certificate signed by a trusted CA. The browser verifies the signature using the CA's public key (pre-installed in the OS/Browser). If an attacker intercepts the connection, they cannot present a valid certificate for the domain because they don't have the private key corresponding to a trusted public key.

**Q: What is the difference between TLS 1.2 and 1.3?**
**A:** TLS 1.3 is faster and more secure. It reduces the handshake from 2 round-trips (2-RTT) to 1 round-trip (1-RTT) or even 0-RTT (Resumption). It also removes older, insecure cryptographic suites (like RC4, SHA-1, CBC mode) that were supported in 1.2.

**Q: Explain SSL Termination vs. SSL Passthrough.**
**A:**
*   **Termination**: The Load Balancer decrypts the traffic, inspects it (L7), and sends unencrypted HTTP to the backend servers. This offloads CPU work from the backends.
*   **Passthrough**: The Load Balancer sends the encrypted TCP stream directly to the backend. The backend must have the certificate and handle decryption. This is more secure (end-to-end encryption) but prevents the LB from inspecting headers.
