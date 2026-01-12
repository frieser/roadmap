---
---

## Summary
SHA (Secure Hash Algorithm) is a family of cryptographic hash functions published by the NIST. The family includes SHA-1, SHA-2 (which includes SHA-256 and SHA-512), and SHA-3.

## Detailed Explanation
### SHA-1
SHA-1 produces a 160-bit hash. Like MD5, it is now considered **insecure** because researchers have successfully demonstrated collision attacks. It should no longer be used for digital signatures or SSL certificates.

### SHA-2
SHA-2 is the current industry standard. It includes several variants based on bit-length:
- **SHA-256**: Extremely common, used in Bitcoin and SSL certificates.
- **SHA-512**: Often used in high-security environments.
SHA-2 is currently considered secure, though it is vulnerable to "length extension attacks" if used incorrectly in certain protocols.

### SHA-3
The latest member of the family, based on a different internal structure (Keccak). It was released in 2015 as an alternative to SHA-2, providing more diversity in hashing standards.

## Go Context
Go provides SHA implementations in the `crypto/sha1`, `crypto/sha256`, and `crypto/sha512` packages.

### Example: SHA-256 in Go
```go
package main

import (
	"crypto/sha256"
	"fmt"
)

func main() {
	h := sha256.New()
	h.Write([]byte("secure data"))
	bs := h.Sum(nil)
	fmt.Printf("%x\n", bs)
}
```

## Interview Questions
- **Q: What is the difference between SHA-1 and SHA-256?**
- **A:** SHA-1 produces a 160-bit hash and is cryptographically broken. SHA-256 produces a 256-bit hash and is currently considered secure for most applications.

- **Q: Is SHA-256 suitable for password hashing?**
- **A:** No. Like SHA-1 and MD5, SHA-256 is designed to be fast. Fast hashes are easy to brute-force. Passwords should be hashed with "slow" algorithms like Bcrypt, Argon2, or Scrypt.

- **Q: What is a "Salt" in the context of hashing?**
- **A:** A salt is random data added to the input before hashing. It ensures that identical passwords result in different hashes, protecting against Rainbow Table attacks.
