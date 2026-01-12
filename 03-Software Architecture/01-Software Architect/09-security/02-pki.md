---
---

## Summary
**Public Key Infrastructure (PKI)** is the comprehensive system of hardware, software, people, policies, and procedures needed to create, manage, distribute, use, store, and revoke **Digital Certificates**. At its core, PKI provides a mechanism for establishing **Trust** in an untrusted environment (like the Internet) by binding public keys to identities (entities like websites, servers, or users) through a hierarchy of trusted third parties called **Certificate Authorities (CA)**.

## Detailed Explanation

### 1. The Foundation: Asymmetric Encryption
PKI is built on **Asymmetric Cryptography** (Public Key Cryptography), which uses a mathematically linked pair of keys:
*   **Public Key**: Can be shared with anyone. Used to encrypt data or verify a digital signature.
*   **Private Key**: Must be kept secret. Used to decrypt data or create a digital signature.
*   **Algorithms**: Common choices include **RSA** (based on prime factorization) and **ECDSA/Ed25519** (based on elliptic curves, providing better security with smaller keys).

### 2. X.509 Certificates
A certificate is a digital document that proves ownership of a public key. The most common standard is **X.509**. It includes:
*   **Subject**: The identity of the owner (e.g., \`CN=example.com\`).
*   **Public Key**: The owner's public key.
*   **Issuer**: The CA that signed the certificate.
*   **Validity Period**: Not Before and Not After dates.
*   **Digital Signature**: The CA's signature of the certificate's hash, proving it hasn't been tampered with.

### 3. The Chain of Trust
Trust is hierarchical. To trust a "Leaf" certificate, you must trust the entity that signed it.
*   **Root CA**: The anchor of trust. It is self-signed and stored in the "Trust Store" of your OS or browser.
*   **Intermediate CA**: Acts as a buffer between the Root and the Leaf. If an intermediate is compromised, the Root remains safe (offline).
*   **Leaf Certificate**: The end-entity certificate (e.g., used by your web server).
*   **Verification**: The client verifies the Leaf using the Intermediate's public key, then verifies the Intermediate using the Root's public key.

### 4. SSL/TLS Handshake & mTLS
The **TLS Handshake** is the process where PKI is used to establish a secure connection:
1.  **Handshake**: Client and Server agree on cipher suites and exchange random numbers.
2.  **Server Authentication**: Server sends its Certificate. Client verifies it against its Trust Store (Chain of Trust).
3.  **Key Exchange**: Parties generate a **Symmetric Key** (AES) for the actual data transfer (asymmetric is too slow for large data).
4.  **Mutual TLS (mTLS)**: In standard TLS, only the server proves its identity. In **mTLS**, the client *also* presents a certificate, and the server verifies it. This is a cornerstone of **Zero Trust** architecture in microservices.

## Go Application
Go's \`crypto/tls\` and \`crypto/x509\` packages provide robust support for PKI and mTLS.

### mTLS Server Implementation
\`\`\`go
package main

import (
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"log"
	"net/http"
	"os"
)

func main() {
	// 1. Load Server Certificate and Private Key
	cert, err := tls.LoadX509KeyPair("server.crt", "server.key")
	if err != nil {
		log.Fatal(err)
	}

	// 2. Load CA certificate to verify Client certificates
	caCert, err := os.ReadFile("ca.crt")
	if err != nil {
		log.Fatal(err)
	}
	caCertPool := x509.NewCertPool()
	caCertPool.AppendCertsFromPEM(caCert)

	// 3. Configure TLS with ClientAuth (mTLS)
	tlsConfig := &tls.Config{
		Certificates: []tls.Certificate{cert},
		ClientAuth:   tls.RequireAndVerifyClientCert,
		ClientCAs:    caCertPool,
	}

	server := &http.Server{
		Addr:      ":443",
		TLSConfig: tlsConfig,
	}

	fmt.Println("mTLS Server running on :443")
	log.Fatal(server.ListenAndServeTLS("", ""))
}
\`\`\`

## Interview Questions
**Q: What is the purpose of an Intermediate CA?**
**A:** Security and risk mitigation. Root CAs are kept offline to prevent compromise. Intermediate CAs are used for day-to-day signing. If an Intermediate is compromised, it can be revoked without needing to replace the Root anchor in millions of devices.

**Q: Explain the difference between CRL and OCSP.**
**A:** Both are used for certificate revocation. **CRL** (Certificate Revocation List) is a list of revoked serial numbers downloaded periodically. **OCSP** (Online Certificate Status Protocol) is a real-time query to a responder to check a specific certificate's status.

**Q: Why do we use Symmetric encryption after the TLS handshake?**
**A:** Performance. Asymmetric encryption (RSA/ECC) is computationally expensive. It is used only to securely exchange a "Session Key." Once the key is shared, Symmetric encryption (AES/ChaCha20) is used because it is much faster and more efficient for bulk data.

**Q: What happens if a Root CA's private key is stolen?**
**A:** The entire Chain of Trust is broken. Any certificate signed by that Root (or its intermediates) can no longer be trusted. The Root must be removed from all Trust Stores worldwide, which is a major security event.

## Diagram
\`\`\`mermaid
sequenceDiagram
    participant Client
    participant Server
    participant CA as Certificate Authority

    Note over Server, CA: Server generates Key Pair
    Server->>CA: Certificate Signing Request (CSR)
    CA-->>Server: Signed X.509 Certificate
    
    Note over Client, Server: TLS Handshake (mTLS)
    Client->>Server: Client Hello
    Server->>Client: Server Hello + Server Certificate
    Client->>Client: Verify Server Cert (Chain of Trust)
    Client->>Server: Client Certificate
    Server->>Server: Verify Client Cert
    Note over Client, Server: Secure Session Established
\`\`\`
