#API
---
---

# Do Not Reinvent the Wheel: API Authentication

## Summary
The principle of **"Do not reinvent the wheel"** is a critical security directive in API Authentication and Cryptography. Cryptography is notoriously difficult to get right; even minor implementation flaws can lead to catastrophic vulnerabilities. By "rolling your own" crypto or authentication logic, you bypass decades of peer review, mathematical proofs, and battle-testing that established libraries provide. Using standard, audited libraries ensures protection against subtle attacks (like timing or side-channel attacks) that are nearly impossible to catch during standard development.

## Detailed Explanation

### 1. The Risks of Custom Implementations
Custom cryptographic or authentication code often suffers from several "silent" failures:
*   **Weak Randomness**: Using non-cryptographic PRNGs (like `math/rand`) instead of CSPRNGs (`crypto/rand`) makes keys and tokens predictable.
*   **Side-Channel Attacks**: Custom comparison logic often terminates early when a character mismatch is found, allowing attackers to guess secrets character-by-character by measuring response times (**Timing Attacks**).
*   **Lack of Agility**: Established libraries allow for easy "cost factor" updates or algorithm migration as hardware improves.
*   **Improper Salting**: Handling salts manually often leads to salt reuse or storing them insecurely, enabling rainbow table attacks.

### 2. Established Go Libraries
Instead of custom logic, Go developers should use the following standard or extended (`x/crypto`) packages:
*   **`golang.org/x/crypto/bcrypt`**: The standard for password hashing. It is intentionally slow (adaptive) and handles salting automatically.
*   **`golang.org/x/crypto/argon2`**: The winner of the Password Hashing Competition. It is memory-hard, making it highly resistant to GPU/ASIC cracking.
*   **`crypto/rand`**: Essential for generating secure tokens, salts, and nonces.
*   **`crypto/subtle`**: Provides functions like `ConstantTimeCompare` to prevent timing attacks.

### 3. Go Example: Correct vs. Incorrect Hashing
A common mistake is using a fast, general-purpose hash (like SHA-256) for passwords. These are designed for file integrity/APIs and are too fast, making them trivial to brute-force.

```go
package main

import (
	"crypto/sha256"
	"fmt"
	"golang.org/x/crypto/bcrypt"
)

func main() {
	password := "user_password_123"

	// --- INCORRECT: Rolling your own "secure" hash with SHA-256 ---
	// Problem: Too fast (billions of attempts/sec on GPU), no built-in salt.
	badHash := sha256.Sum256([]byte(password))
	fmt.Printf("Bad (SHA256): %x\n", badHash)

	// --- CORRECT: Using established bcrypt library ---
	// Benefit: Computationally expensive (slows brute force), automatic salt.
	cost := 12 // Adaptive work factor
	goodHash, err := bcrypt.GenerateFromPassword([]byte(password), cost)
	if err != nil {
		panic(err)
	}
	fmt.Printf("Good (Bcrypt): %s\n", string(goodHash))

	// Verification is also handled by the library
	err = bcrypt.CompareHashAndPassword(goodHash, []byte(password))
	if err == nil {
		fmt.Println("Authentication Successful!")
	}
}
```

## Interview Questions

**Q: Why is "rolling your own crypto" considered one of the biggest sins in security?**
**A:** Cryptographic algorithms are secure not just because of the math, but because of the specific implementation details that prevent side-channel attacks (like timing or power analysis). Standard libraries like OpenSSL or Go's `crypto` package are audited by experts. A developer's custom implementation might be mathematically sound but leak secrets through response times or memory patterns.

**Q: What is the difference between `math/rand` and `crypto/rand` in Go?**
**A:** `math/rand` is a pseudo-random number generator (PRNG) that is deterministic if you know the seed. It is fast but predictable, making it unsuitable for security. `crypto/rand` uses the operating system's cryptographically secure random number generator (CSPRNG), providing true entropy required for keys, salts, and tokens.

**Q: How do you prevent timing attacks when comparing sensitive tokens in Go?**
**A:** You should use `crypto/subtle.ConstantTimeCompare(a, b []byte)`. Standard string or byte comparisons (`a == b`) return as soon as a difference is found. An attacker can measure this time difference to determine how many characters of their guess were correct. `ConstantTimeCompare` always takes the same amount of time regardless of whether the inputs match.

**Q: Why should you prefer Argon2 over Bcrypt for new systems?**
**A:** While Bcrypt is still secure, Argon2 is "memory-hard." It requires a specific amount of RAM to compute, which makes it much harder to crack using specialized hardware like ASICs or GPUs, which have plenty of processing power but limited memory per core compared to a CPU.
