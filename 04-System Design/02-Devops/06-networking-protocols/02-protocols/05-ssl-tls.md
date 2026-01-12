---
---

# SSL/TLS (Secure Sockets Layer / Transport Layer Security)

While often used interchangeably, SSL is the deprecated predecessor, and TLS is the modern standard protocol for securing communication over a computer network. It provides privacy and data integrity between two communicating applications.

## Summary

TLS sits between the Application Layer (L7) and the Transport Layer (L4). It wraps protocols like HTTP, SMTP, and FTP to create HTTPS, SMTPS, and FTPS. The protocol consists of two phases: the **Handshake Protocol** (negotiating keys and verifying identity) and the **Record Protocol** (encrypting data with shared keys).

## Detailed Explanation

### 1. The TLS Handshake (Simplified)
1.  **Client Hello**: Client sends supported cipher suites and a random number.
2.  **Server Hello**: Server picks a cipher suite, sends its Certificate, and a random number.
3.  **Key Exchange**:
    *   **RSA**: Client generates a "Pre-Master Secret", encrypts it with Server's Public Key, and sends it.
    *   **Diffie-Hellman (DHE/ECDHE)**: Both parties generate keys to independently calculate the shared secret (Perfect Forward Secrecy).
4.  **Finished**: Both switch to symmetric encryption using the generated session keys.

### 2. Certificates & PKI
*   **X.509**: The standard format for public key certificates.
*   **Components**: Subject (Domain), Issuer (CA), Public Key, Signature, Validity Period.
*   **Revocation**: Checked via **CRL** (Certificate Revocation List) or **OCSP** (Online Certificate Status Protocol).

### 3. Mutual TLS (mTLS)
In standard TLS, only the server proves its identity. In **mTLS**, the client *also* has a certificate and proves its identity to the server. This is common in **Zero Trust** architectures and Service Meshes (like Istio) to secure service-to-service communication.

---

## Go Implementation Example

Go's `crypto/tls` package allows for fine-grained control over the TLS configuration, including loading certificates and enforcing mTLS.

```go
package main

import (
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"log"
	"os"
)

func main() {
	// 1. Load Server Certificate and Key
	cert, err := tls.LoadX509KeyPair("server.crt", "server.key")
	if err != nil {
		log.Fatal("Failed to load keys: ", err)
	}

	// 2. Load CA Cert to verify clients (for mTLS)
	caCert, _ := os.ReadFile("ca.crt")
	caCertPool := x509.NewCertPool()
	caCertPool.AppendCertsFromPEM(caCert)

	// 3. Configure TLS
	config := &tls.Config{
		Certificates: []tls.Certificate{cert},
		// Enforce mTLS: Client MUST present a valid cert
		ClientAuth: tls.RequireAndVerifyClientCert,
		ClientCAs:  caCertPool,
		MinVersion: tls.VersionTLS13, // Enforce modern security
	}

	// 4. Create Listener
	listener, err := tls.Listen("tcp", ":8443", config)
	if err != nil {
		log.Fatal(err)
	}
	defer listener.Close()

	fmt.Println("mTLS Server listening on :8443")
	
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Println(err)
			continue
		}
		// Connection is already encrypted and authenticated!
		conn.Write([]byte("Hello, Authenticated Client!\n"))
		conn.Close()
	}
}
```

## Interview Questions

**Q: What is "Perfect Forward Secrecy" (PFS)?**
**A:** PFS ensures that even if the server's private key is compromised in the future, past sessions cannot be decrypted. This is achieved using ephemeral key exchange algorithms (like **ECDHE**) where a unique session key is generated for every connection and never stored. RSA key exchange does *not* support PFS.

**Q: Why is disabling TLS 1.0/1.1 recommended?**
**A:** TLS 1.0 (1999) and 1.1 (2006) have known cryptographic vulnerabilities (BEAST, POODLE, etc.) and support weak hashing algorithms like SHA-1 and MD5. Modern compliance standards (PCI-DSS) require disabling them in favor of TLS 1.2+.

**Q: What happens if a certificate expires?**
**A:** The browser/client will terminate the connection and display a security warning ("NET::ERR_CERT_DATE_INVALID"). In automated DevOps environments, this causes service outages. This is why automated certificate management (e.g., **cert-manager** in Kubernetes) is critical.
