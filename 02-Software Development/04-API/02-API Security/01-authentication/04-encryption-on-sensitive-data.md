#API
---
---

# Encryption on Sensitive Data

## Summary
Encryption on Sensitive Data ensures that information remains unreadable to unauthorized parties, whether it is being transmitted over a network (**Data in Transit**) or stored on disk (**Data at Rest**).
*   **Data in Transit**: Secured via **TLS (Transport Layer Security)**. This prevents eavesdropping and tampering between the client and the server.
*   **Data at Rest**: Secured via **Symmetric Encryption** (e.g., AES-GCM) or **Asymmetric Encryption** (e.g., RSA). This protects data if the database or physical storage media is compromised.

## Detailed Explanation

### 1. Key Management (KMS / Vault)
The strength of encryption relies entirely on the secrecy of the keys.
*   **Bad Practice**: Hardcoding keys in source code (`const key = "12345"`).
*   **Good Practice**: Using Environment Variables or Secrets Managers (AWS KMS, HashiCorp Vault).
*   **Rotation**: Regularly changing encryption keys (Key Rotation) limits the "blast radius" if a key is ever leaked.

### 2. Hashing vs. Encryption
*   **Hashing** (e.g., Argon2, Bcrypt): One-way transformation. Use for **Passwords**. You cannot retrieve the original data.
*   **Encryption** (e.g., AES): Reversible transformation (with a key). Use for **PII** (SSN, Email, Address) where the application needs to read the data later.

### 3. Encryption Flow (Mermaid)

```mermaid
graph TD
    User[User Input (SSN)] --> API[Go API Service]
    API --> KMS[Key Management Service]
    KMS -->|Fetch Data Key| API
    API -->|Encrypt (AES-GCM)| Ciphertext[Encrypted Data]
    Ciphertext --> DB[(Database)]
    
    subgraph "Decryption"
    DB -->|Read Ciphertext| API
    API -->|Decrypt (AES-GCM)| Plaintext
    end
```

### 4. Go Example: AES-GCM Encryption
Go's `crypto/cipher` package provides authenticated encryption. **AES-GCM** (Galois/Counter Mode) is the standard because it provides both confidentiality (encryption) and integrity (it detects tampering).

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"encoding/hex"
	"fmt"
	"io"
)

func encrypt(plaintext []byte, key []byte) (string, error) {
	// 1. Create Cipher Block
	block, err := aes.NewCipher(key)
	if err != nil {
		return "", err
	}

	// 2. Create GCM (Galois Counter Mode) - Standard for Authenticated Encryption
	aesGCM, err := cipher.NewGCM(block)
	if err != nil {
		return "", err
	}

	// 3. Create Nonce (Number used once)
	nonce := make([]byte, aesGCM.NonceSize())
	if _, err = io.ReadFull(rand.Reader, nonce); err != nil {
		return "", err
	}

	// 4. Encrypt and Seal (Nonce + Ciphertext + Tag)
	ciphertext := aesGCM.Seal(nonce, nonce, plaintext, nil)
	return hex.EncodeToString(ciphertext), nil
}

func main() {
	key := []byte("example key 12345678901234567890") // 32 bytes for AES-256
	ssn := []byte("123-45-6789")

	encrypted, _ := encrypt(ssn, key)
	fmt.Printf("Stored in DB: %s\n", encrypted)
}
```

## Interview Questions

1.  **Why is AES-GCM preferred over AES-CBC?**
    *   *Answer:* AES-GCM provides **Authenticated Encryption with Associated Data (AEAD)**. It ensures both confidentiality (encryption) and integrity (tamper detection) in a single pass. AES-CBC is malleable (susceptible to padding oracle attacks) unless combined with a separate MAC (Message Authentication Code) like HMAC.

2.  **What happens if you reuse a Nonce in AES-GCM?**
    *   *Answer:* Reusing a nonce with the same key allows an attacker to XOR the ciphertexts to recover the XOR of the plaintexts, potentially breaking the encryption completely. Nonces must be unique for every encryption operation.

3.  **How does TLS protect Data in Transit?**
    *   *Answer:* TLS performs a "Handshake" using asymmetric cryptography (public/private keys) to securely exchange a shared secret. It then uses this shared secret for symmetric encryption (like AES) for the rest of the session, ensuring high performance and security.

4.  **When should you Hash data instead of Encrypting it?**
    *   *Answer:* Hash data when you never need to see the original value again, only verify it (e.g., Passwords). Encrypt data when the business logic requires the original value (e.g., sending an email to a user, processing a credit card transaction).
