---
---

## Summary
Hashing algorithms are one-way mathematical functions that transform input data of any size into a fixed-length output (hash). In computer science, they are divided into **General Purpose** (for speed and data structures) and **Cryptographic** (for security and integrity). For password storage, algorithms must be intentionally slow to resist brute-force attacks, utilizing concepts like **Salts**, **Peppers**, and **Work Factors**.

## Detailed Explanation

### 1. Categories of Hashing Algorithms

#### General Purpose (Non-Cryptographic)
These are optimized for speed and are used in performance-critical applications where security is not a concern.
- **Usage**: Hash tables (maps), checksums for data integrity (non-malicious), and deduplication.
- **Common Algorithms**: 
    - **MurmurHash**: High performance and low collision rate, common in distributed systems like Apache Cassandra or Redis.
    - **CRC32**: Used in ZIP files and Ethernet for detecting accidental changes in data.

#### Cryptographic Hashing
These must satisfy specific security properties: one-wayness (pre-image resistance), collision resistance, and the **Avalanche Effect** (small input change = massive output change).
- **Usage**: Digital signatures, message integrity, and password verification.
- **Common Algorithms**:
    - **MD5 / SHA-1**: Legacy algorithms, now considered **broken** due to collision vulnerabilities.
    - **SHA-2 (SHA-256, SHA-512)**: The current industry standard for general cryptographic needs (SSL/TLS, Bitcoin).
    - **SHA-3**: The latest member of the Secure Hash Algorithm family, based on the Keccak construction.

---

### 2. Password Hashing: Why Speed is Bad
A common mistake is using fast cryptographic hashes (like SHA-256) for passwords. A modern GPU can compute billions of SHA-256 hashes per second, making brute-force or rainbow table attacks trivial.

**Key Security Concepts:**
- **Work Factor (Cost)**: A parameter that increases the computational power (CPU, memory, or time) required to calculate a single hash. This scales the difficulty for attackers.
- **Salt**: A random string added to the password before hashing. It ensures that two users with the same password have different hashes, rendering **Rainbow Tables** (pre-computed hash lists) useless.
- **Pepper**: A secret value added to the password, similar to a salt, but stored separately from the database (e.g., in an environment variable or HSM).

---

### 3. Recommended Algorithms for Passwords
- **Bcrypt**: Adaptive hash function based on the Blowfish cipher. It includes a built-in salt and a configurable "cost" factor.
- **Argon2 (Argon2id)**: The winner of the Password Hashing Competition (2015). It is resistant to GPU-based and side-channel attacks by being "memory-hard." This is the modern **gold standard**.

---

### 4. Go Implementation
Go provides the `bcrypt` implementation in the `golang.org/x/crypto` sub-repository.

```go
package main

import (
	"fmt"
	"golang.org/x/crypto/bcrypt"
)

// HashPassword generates a bcrypt hash of the password using the default cost (10)
func HashPassword(password string) (string, error) {
	bytes, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
	return string(bytes), err
}

// CheckPasswordHash compares a bcrypt hashed password with its possible plaintext equivalent
func CheckPasswordHash(password, hash string) bool {
	err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
	return err == nil
}

func main() {
	password := "my_secret_password_123"
	
	// 1. Hash the password
	hash, _ := HashPassword(password)
	fmt.Println("Password:", password)
	fmt.Println("Hash:    ", hash)

	// 2. Verify the password
	match := CheckPasswordHash(password, hash)
	fmt.Println("Match:   ", match)
    
	// 3. Verify with wrong password
	matchWrong := CheckPasswordHash("wrong_password", hash)
	fmt.Println("Match Wrong:", matchWrong)
}
```

---

## Interview Questions

**Q: What is the difference between Hashing and Encryption?**
**A:** Hashing is a **one-way** function; you cannot retrieve the original data from the hash. Encryption is **two-way**; data can be decrypted back to its original form using a key.

**Q: Why is SHA-256 not recommended for password storage?**
**A:** SHA-256 is designed to be **fast**. While secure for integrity, its speed allows attackers to perform high-speed brute-force attacks. Password hashes should be "slow" (computationally expensive).

**Q: What is a Salt and why is it used?**
**A:** A salt is random data added to a password before hashing. It prevents attackers from using pre-computed hash tables (Rainbow Tables) and ensures that identical passwords result in different hashes.

**Q: What is "Memory-Hardness" in the context of Argon2?**
**A:** It means the algorithm requires a significant amount of RAM to compute. This makes it difficult and expensive to attack using specialized hardware like ASICs or GPUs, which excel at computation but have limited high-speed memory per core.

**Q: How do you handle a change in your "Work Factor" or hashing algorithm?**
**A:** You implement **re-hashing on login**. When a user logs in, verify their old hash; if successful, immediately hash the plaintext password using the new algorithm/cost and update the database record.
