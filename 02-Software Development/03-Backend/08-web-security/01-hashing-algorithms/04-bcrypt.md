---
---

## Summary
Bcrypt is a password-hashing function based on the Blowfish cipher. It is designed to be slow and computationally expensive to prevent brute-force attacks. It includes a salt to protect against rainbow table attacks and an adjustable "cost" factor to stay secure as hardware improves.

## Detailed Explanation
Bcrypt was presented in 1999 and remains one of the most recommended algorithms for password storage.

### Key Features
- **Built-in Salt**: The salt is automatically generated and stored inside the final hash string. You don't need a separate database column for the salt.
- **Cost Factor (Work Factor)**: An integer that determines how many iterations of the hashing algorithm are performed. Increasing the cost by 1 doubles the time it takes to compute the hash.
- **Pre-set Limit**: Bcrypt has a 72-byte limit on the password length. Longer passwords are truncated.

### Format of a Bcrypt Hash
Example: `$2a$12$R9h/lSAbvIogwQdEwXycKuBbO94z7zWfJpzwE3/f.W.b.R.W.b.R.W`
- `$2a$`: The algorithm version.
- `$12$`: The cost factor ($2^{12}$ iterations).
- The rest is the salt and the actual hash combined.

## Go Context
Go provides Bcrypt in the `golang.org/x/crypto/bcrypt` package.

### Example: Hashing and Comparing Passwords in Go
```go
package main

import (
	"fmt"
	"golang.org/x/crypto/bcrypt"
)

func main() {
	password := "mypassword123"

	// Hash password with default cost (10)
	hash, _ := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
	fmt.Println("Hash:", string(hash))

	// Compare password with hash
	err := bcrypt.CompareHashAndPassword(hash, []byte(password))
	if err == nil {
		fmt.Println("Password matches!")
	} else {
		fmt.Println("Invalid password")
	}
}
```

## Interview Questions
- **Q: Why is Bcrypt better than SHA-256 for passwords?**
- **A:** SHA-256 is a general-purpose hash designed for speed. Bcrypt is specifically designed for passwords and is intentionally slow. It also has built-in salt handling and an adjustable cost factor.

- **Q: What is the 72-character limit in Bcrypt?**
- **A:** Bcrypt only considers the first 72 bytes of a password. To work around this for very long passwords, some developers hash the password with SHA-256 first and then Bcrypt the resulting SHA hash.

- **Q: How do you decide on the "Cost Factor"?**
- **A:** You should set the cost factor as high as possible while still maintaining a reasonable user experience (typically aiming for 100ms to 500ms for a single login request on your production hardware).
