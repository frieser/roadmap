#API #Security #JWT
---
---

# Do Not Extract Algorithm From Header

## Summary
The **"Do Not Extract Algorithm From Header"** rule mandates that the server must decide which algorithm to use for verification based on its own configuration, rather than trusting the `alg` field provided in the token's header. Violating this rule exposes the system to critical vulnerabilities like the **"None" Algorithm Attack** and **Key Confusion Attacks**.

## Detailed Explanation

### 1. The Vulnerability
JWTs are self-describing; the header contains an `alg` field (e.g., `{"alg": "HS256"}`).
*   **The Flaw**: If a backend blindly trusts this field, an attacker can modify the header to dictate how the token is verified.
*   **The Risk**: Complete authentication bypass.

### 2. The "None" Algorithm Attack (CVE-2015-9235)
*   **Mechanism**: The attacker modifies the header to `{"alg": "none"}` and strips the signature.
*   **Impact**: If the library or implementation supports "none" (often used for debugging) and trusts the header, it will treat the unsigned token as valid. The attacker can then modify the payload (e.g., `user_id: 1`) arbitrarily.

### 3. Key Confusion Attack (HMAC vs. RSA)
*   **Scenario**: The server expects an **RS256** token (signed with a Private Key, verified with a Public Key).
*   **Attack**: The attacker switches the header to `{"alg": "HS256"}` and signs the token using the server's **Public Key** as the HMAC secret.
*   **Exploit**: If the server trusts the `alg` header, it switches to HMAC verification mode. It then uses its configured "verification key" (which is the Public Key) as the shared secret. Since the attacker also has the Public Key, they can forge a valid signature.

### 4. Best Practices
*   **Whitelist Algorithms**: Explicitly configure the server to accept *only* the expected algorithm (e.g., `RS256`).
*   **Hardcode Verification**: Ignore the header's `alg` value during the verification setup phase.
*   **Use Strict Libraries**: Modern libraries (like `golang-jwt`) require a `Keyfunc` callback that forces you to validate the algorithm type.

---

## Go (Golang) Application

### Secure Verification Pattern
Using `golang-jwt/jwt/v5`.

```go
package main

import (
	"fmt"
	"os"

	"github.com/golang-jwt/jwt/v5"
)

func VerifyToken(tokenString string) (*jwt.Token, error) {
	// The key we expect to use for verification
	hmacSecret := []byte(os.Getenv("JWT_SECRET"))

	token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
		// CRITICAL: Validate the algorithm type matches what you expect.
		// If the token says "none" or "RS256", this check will fail.
		if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
			return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
		}

		// Only return the key if the algorithm is correct
		return hmacSecret, nil
	})

	if err != nil {
		return nil, err
	}

	if !token.Valid {
		return nil, fmt.Errorf("invalid token")
	}

	return token, nil
}
```

---

## Interview Questions

**Q1: Explain the mechanism of the JWT "None" algorithm attack.**
**A**: The attacker changes the header to `{"alg": "none"}` and removes the signature. If the backend library trusts the header, it skips the signature verification step, accepting whatever payload the attacker provides as authentic.

**Q2: How does a Key Confusion Attack allow an attacker to forge tokens using a Public Key?**
**A**: The attacker forces the server to use symmetric verification (HS256) instead of asymmetric (RS256). Since the server uses its configured key (the RSA Public Key) as the HMAC secret, and the Public Key is available to the attacker, the attacker can use that same Public Key to sign a forged token using HMAC-SHA256.

**Q3: In Go's `jwt-go` / `golang-jwt` library, what is the purpose of the `Keyfunc` callback?**
**A**: The `Keyfunc` allows the developer to dynamically select the verification key based on the token. Crucially, it provides access to the unverified token, allowing the developer to inspect and validate the `token.Method` (algorithm) *before* returning the key, preventing algorithm substitution attacks.
