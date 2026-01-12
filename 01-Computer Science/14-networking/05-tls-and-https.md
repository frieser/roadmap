---
---

# TLS and HTTPS

**TLS (Transport Layer Security)** is the cryptographic protocol that secures communication over a network. **HTTPS** is simply HTTP running over TLS. It provides three core security pillars: **Confidentiality** (Encryption), **Integrity** (Anti-tampering), and **Authentication** (Identity verification).

---

## 1. Cryptography Basics

TLS uses a hybrid approach: **Asymmetric** encryption for the handshake (securely sharing a secret) and **Symmetric** encryption for the actual data transfer (fast and efficient).

### **Symmetric Encryption**
Uses the same key for both encryption and decryption.
- **AES (Advanced Encryption Standard)**: The industry standard. Uses block sizes (128, 192, 256 bits).
- **ChaCha20**: A high-speed stream cipher, often paired with Poly1305 for integrity. Preferred on mobile devices without hardware AES acceleration.

### **Asymmetric Encryption (Public-Key)**
Uses a pair of keys: a **Public Key** (to encrypt) and a **Private Key** (to decrypt).
- **RSA**: Relies on the difficulty of factoring large prime numbers. Requires large keys (2048+ bits).
- **ECC (Elliptic Curve Cryptography)**: Relies on elliptic curves. Provides same security as RSA with much smaller keys (e.g., 256-bit ECC ≈ 3072-bit RSA). Used in modern TLS (ECDHE).

### **Hashing**
One-way functions that turn data into a fixed-size string (fingerprint). Used for data integrity.
- **SHA-256 / SHA-512**: Current standard for certificates and message authentication (HMAC).

---

## 2. The TLS Handshake

The handshake establishes a secure session by negotiating protocols, authenticating the server, and generating session keys.

### **TLS 1.2 Handshake (2-RTT)**
Requires two round trips before data can be sent.
1.  **ClientHello**: Supported versions, cipher suites, and a random number ($R_c$).
2.  **ServerHello**: Selected version, cipher, and a random number ($R_s$).
3.  **Server Certificate**: Server sends its public key and CA-signed cert.
4.  **Client Key Exchange**: Client sends a "Pre-Master Secret" (PMS) encrypted with server's public key.
5.  **Change Cipher Spec**: Both derive the **Session Key** from $R_c$, $R_s$, and PMS.

### **TLS 1.3 Handshake (1-RTT)**
Faster and more secure (released in 2018).
1.  **ClientHello**: Client guesses the server's key exchange algorithm and sends its **Key Share** immediately.
2.  **ServerHello**: Server sends its **Key Share**, Certificate, and "Finished" message.
3.  **Data**: Encryption begins immediately after 1-RTT.
4.  **0-RTT Support**: Allows returning clients to send data in the very first packet using a "resumption secret."

---

## 3. Public Key Infrastructure (PKI)

PKI is the system of hardware, software, and people that manage digital certificates.

- **Certificate Authority (CA)**: A trusted third party (e.g., Let's Encrypt, DigiCert) that verifies identities and issues digital certificates.
- **Chain of Trust**:
    1.  **Root Certificate**: Self-signed by the CA. Pre-installed in OS/Browser **Root Stores**.
    2.  **Intermediate Certificate**: Signed by the Root; used to sign end-entity certs (improves security by keeping the Root offline).
    3.  **Leaf (End-Entity) Certificate**: Your website's certificate.
- **Root Store**: A collection of trusted Root CAs maintained by OS (Windows/macOS) or browser (Firefox) vendors. If a cert isn't signed by a CA in your Root Store, you see a "Not Secure" warning.

---

## 4. Go Implementation: `crypto/tls`

In Go, loading a certificate and starting a secure server is handled by `net/http` and `crypto/tls`.

**Example: Loading Cert/Key Pair**
```go
package main

import (
	"crypto/tls"
	"log"
	"net/http"
)

func main() {
	// Load certificate and private key from files
	cert, err := tls.LoadX509KeyPair("server.crt", "server.key")
	if err != nil {
		log.Fatalf("Failed to load keys: %s", err)
	}

	// Configure TLS
	tlsConfig := &tls.Config{
		Certificates: []tls.Certificate{cert},
		MinVersion:   tls.VersionTLS12, // Baseline security
	}

	server := &http.Server{
		Addr:      ":443",
		TLSConfig: tlsConfig,
	}

	log.Println("Starting HTTPS server...")
	log.Fatal(server.ListenAndServeTLS("", "")) // Files already in config
}
```

**Evidence** ([grpc-go/internal/testutils/tls_creds.go](https://github.com/grpc/grpc-go/blob/master/internal/testutils/tls_creds.go#L37)):
```go
cert, err := tls.LoadX509KeyPair(testdata.Path("x509/client1_cert.pem"), testdata.Path("x509/client1_key.pem"))
```

---

## 5. Interview Questions

**Q1: What is the difference between Symmetric and Asymmetric encryption in TLS?**
**A:** Asymmetric encryption (RSA/ECC) is used during the handshake to securely exchange a secret because it doesn't require a pre-shared key. Symmetric encryption (AES) is used for the actual data transfer because it is orders of magnitude faster and less CPU-intensive.

**Q2: How does TLS 1.3 improve upon TLS 1.2?**
**A:** 1) **Speed**: Reduces the handshake from 2-RTT to 1-RTT (and supports 0-RTT for resumption). 2) **Security**: Removes legacy/weak algorithms (MD5, SHA-1, RC4, RSA key transport) and mandates Perfect Forward Secrecy (PFS) via Diffie-Hellman.

**Q3: What happens if a Root CA is compromised?**
**A:** If a Root CA's private key is leaked, attackers can issue valid-looking certificates for any domain. To fix this, OS/Browser vendors must remove the compromised Root CA from their **Root Stores** via security updates, and all certificates in that chain must be revoked.

**Q4: What is SNI (Server Name Indication)?**
**A:** An extension to the TLS protocol that allows the client to indicate the hostname it is trying to connect to at the start of the handshake (in the `ClientHello`). This allows a single IP address to host multiple HTTPS sites with different certificates.

**Q5: What is mTLS (Mutual TLS)?**
**A:** In standard TLS, only the server proves its identity. In **mTLS**, the client *also* presents a certificate, and the server verifies it. This is commonly used for service-to-service communication in microservices and Zero Trust architectures.

---
## References
- [[Work/Search/Roadmap/01-Computer Science/19-security/02-public-key-cryptography.md|Public Key Cryptography Deep Dive]]
- [[Work/Search/Roadmap/03-Software Architecture/01-Software Architect/14-networks/03-http-https.md|HTTP vs HTTPS Comparison]]
