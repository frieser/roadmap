---
---

## Summary
**Public Key Cryptography (PKC)**, also known as **Asymmetric Encryption**, is a cryptographic system that uses pairs of keys: **Public Keys** which may be disseminated widely, and **Private Keys** which are known only to the owner. It provides the foundation for secure communication on the internet (TLS/SSL), digital identities, and blockchain technology.

## Core Concepts

### 1. Asymmetric Key Pairs
Unlike Symmetric encryption (where one key is used for both encryption and decryption), PKC uses two mathematically related but distinct keys:
*   **Public Key**: Used by others to encrypt data for you or to verify your signatures.
*   **Private Key**: Used by you to decrypt data sent to you or to create digital signatures. It must never be shared.

### 2. Mathematical Trapdoors
The security of PKC relies on **one-way functions with a trapdoor**. These are mathematical operations that are easy to perform in one direction but extremely difficult to reverse without a specific piece of information (the "trapdoor" or private key).

| Algorithm | Mathematical Trapdoor | Complexity Basis |
| :--- | :--- | :--- |
| **RSA** | **Integer Factorization** | Multiplying two large primes is easy; factoring the result is computationally "hard". |
| **ECC** | **Elliptic Curve Discrete Logarithm Problem (ECDLP)** | Finding the scalar $k$ such that $Q = kP$ on an elliptic curve is extremely difficult. |

## Algorithms: RSA vs ECC

| Feature | RSA (Rivest-Shamir-Adleman) | ECC (Elliptic Curve Cryptography) |
| :--- | :--- | :--- |
| **Security Basis** | Prime Factorization | Discrete Logarithms on Curves |
| **Key Size (128-bit security)** | 3072 bits | 256 bits |
| **Key Size (256-bit security)** | 15360 bits | 512 bits |
| **Performance** | Fast verification, slow signing/decryption | Faster signing, smaller overhead |
| **Efficiency** | High storage/bandwidth requirement | Highly efficient; ideal for mobile/IoT |

**The Verdict**: ECC is the modern standard for most applications (TLS 1.3, Bitcoin) because it offers equivalent security to RSA with significantly smaller keys and less computational power.

## Application Logic

### 1. Encryption (Confidentiality)
*   **Goal**: Ensure only the recipient can read the message.
*   **Process**:
    1.  Sender retrieves Recipient's **Public Key**.
    2.  Sender encrypts the message with the **Public Key**.
    3.  Recipient decrypts the message with their **Private Key**.

### 2. Digital Signatures (Authenticity & Integrity)
*   **Goal**: Prove the sender's identity and that the message hasn't been tampered with.
*   **Process**:
    1.  Sender creates a hash of the message.
    2.  Sender encrypts the hash with their **Private Key** (this is the Signature).
    3.  Recipient decrypts the signature with the Sender's **Public Key**.
    4.  Recipient compares the result with a fresh hash of the received message.

## Go Implementation

This example demonstrates generating an **ECC** key pair (using P-256) and signing a message, as well as an **RSA** example.

```go
package main

import (
	"crypto"
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha256"
	"fmt"
	"log"
)

func main() {
	message := []byte("The eagle has landed.")
	hash := sha256.Sum256(message)

	// --- ECC (Recommended) ---
	fmt.Println("--- ECC Example ---")
	// 1. Generate Key Pair
	eccPriv, _ := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
	eccPub := &eccPriv.PublicKey

	// 2. Sign (using ASN.1 encoding)
	sig, err := ecdsa.SignASN1(rand.Reader, eccPriv, hash[:])
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("ECC Signature: %x\n", sig)

	// 3. Verify
	valid := ecdsa.VerifyASN1(eccPub, hash[:], sig)
	fmt.Printf("ECC Signature Valid: %v\n\n", valid)

	// --- RSA (Standard) ---
	fmt.Println("--- RSA Example ---")
	// 1. Generate Key Pair
	rsaPriv, _ := rsa.GenerateKey(rand.Reader, 2048)
	rsaPub := &rsaPriv.PublicKey

	// 2. Sign (using PSS - Probabilistic Signature Scheme)
	rsaSig, _ := rsa.SignPSS(rand.Reader, rsaPriv, crypto.SHA256, hash[:], nil)
	fmt.Printf("RSA Signature: %x...\n", rsaSig[:16])

	// 3. Verify
	err = rsa.VerifyPSS(rsaPub, crypto.SHA256, hash[:], rsaSig, nil)
	if err == nil {
		fmt.Println("RSA Signature Valid: true")
	}
}
```

## Interview Questions

**1. Q: Why is ECC preferred over RSA in modern mobile applications?**
**A**: ECC provides the same level of security with much smaller key sizes (e.g., 256-bit ECC $\approx$ 3072-bit RSA). This leads to faster computation, lower power consumption, and less network bandwidth, which is critical for battery-powered devices.

**2. Q: What is a "Hybrid Cryptosystem" and why do we use it?**
**A**: PKC is computationally slow. In practice (like TLS/HTTPS), we use **Asymmetric Encryption** to securely exchange a temporary "Session Key", and then use **Symmetric Encryption** (like AES) with that key for the actual data transfer, combining security and speed.

**3. Q: Can you decrypt a message with a Public Key?**
**A**: Technically, in some algorithms like RSA, the math works both ways, but conceptually "decryption" is performed with the **Private Key** to restore confidentiality. Using a Public Key to "unlock" something is called **Verification** (in Digital Signatures).

**4. Q: What happens if your Private Key is compromised?**
**A**: The security of all communications encrypted for you is lost, and the attacker can impersonate you by signing messages. You must revoke your certificate via a **Certificate Revocation List (CRL)** or **OCSP** and generate a new key pair.

**5. Q: Explain the concept of Forward Secrecy.**
**A**: Forward Secrecy ensures that if a long-term Private Key is compromised in the future, past session keys remain secure. This is achieved by using the long-term key only for authentication and using an ephemeral key exchange (like Diffie-Hellman) for the actual encryption.
