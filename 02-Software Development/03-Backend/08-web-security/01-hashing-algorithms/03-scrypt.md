---
---

## Summary
Scrypt is a password-based key derivation function (KDF) designed to be intentionally resource-intensive to thwart hardware-accelerated brute-force attacks (like those using custom ASICs or GPUs).

## Detailed Explanation
Designed by Colin Percival in 2009, Scrypt's main innovation is that it is **memory-hard**.

### Why Memory-Hardness Matters
Most hashing algorithms (like SHA) are CPU-intensive. Attackers can build custom hardware (ASICs) that perform millions of these hashes per second very cheaply. Scrypt requires a large amount of RAM to compute a single hash. Since RAM is expensive and hard to scale on a custom chip, it makes hardware attacks much less effective.

### Parameters
Scrypt uses several parameters to tune its complexity:
- **N**: CPU/memory cost parameter (must be a power of 2).
- **r**: Block size parameter.
- **p**: Parallelization parameter.

## Go Context
Go provides Scrypt in the `golang.org/x/crypto/scrypt` package.

### Example: Hashing a password with Scrypt in Go
```go
package main

import (
	"fmt"
	"golang.org/x/crypto/scrypt"
)

func main() {
	password := []byte("user-password")
	salt := []byte("random-salt")
	
	// N=16384, r=8, p=1 are common recommended values
	dk, err := scrypt.Key(password, salt, 16384, 8, 1, 32)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Printf("Hash: %x\n", dk)
}
```

## Interview Questions
- **Q: What makes Scrypt different from Bcrypt?**
- **A:** While both are slow, Scrypt is designed to be memory-hard, whereas Bcrypt is primarily CPU-hard. Scrypt is more resistant to ASIC-based attacks because of its memory requirements.

- **Q: What does the "parallelization parameter (p)" do in Scrypt?**
- **A:** It allows the computation to be split across multiple threads. However, increasing `p` increases the cost for both the defender and the attacker equally, unlike `N` which increases the cost exponentially for certain attack hardware.

- **Q: Why is "slowness" a feature in password hashing?**
- **A:** A slow hash makes brute-force and dictionary attacks computationally expensive. If a server takes 100ms to verify a password, an attacker trying billions of combinations will be significantly slowed down.
