#API #Security #JWT
---
---

# Avoid Sensitive Data in Payload

## Summary
A JSON Web Token (JWT) is **encoded**, not **encrypted**. This means the payload is Base64Url encoded and can be trivially decoded by anyone who intercepts the token (using browser tools or `jwt.io`). Storing sensitive data like PII (Personally Identifiable Information), passwords, or extensive permissions in the payload violates privacy regulations (GDPR) and exposes the application's internal structure to attackers.

## Detailed Explanation

### 1. Encoding vs. Encryption
*   **JWS (Signed)**: The standard "JWT" ensures **Integrity**. You know the data hasn't changed, but everyone can see it. It is like a postcard with a wax seal.
*   **JWE (Encrypted)**: Ensures **Confidentiality**. The data is encrypted and readable only by the holder of the private key.
*   **The Risk**: Developers often mistake Base64 encoding for encryption. `eyJ1c2VyIjoiYWRtaW4ifQ` looks like gibberish, but it decodes instantly to `{"user":"admin"}`.

### 2. Privacy Risks
*   **GDPR/Privacy**: Storing emails, phone numbers, or addresses in a token that is cached in browsers (`localStorage`) or logged by intermediaries (proxies) is a data leak.
*   **Security Through Obscurity**: Hiding internal database IDs or detailed permission logic in the token gives attackers a roadmap of your system.
*   **Replay Attacks**: If a token containing PII is leaked, the data is compromised forever, even if the token expires.

### 3. Payload Best Practices
The payload should only contain the minimum data needed to **identify** the user and **validate** the token.

**Safe Claims:**
*   `sub` (Subject): An opaque identifier (e.g., UUID `550e8400-e29b...`).
*   `iss` (Issuer): The auth server URL.
*   `exp` (Expiration): Timestamp.
*   `jti` (JWT ID): Unique ID for revocation.
*   `aud` (Audience): Who the token is for.

**Unsafe Claims:**
*   `email`: "user@example.com" (PII).
*   `role_details`: "admin_level_5_rw" (Exposes security model).
*   `password_hash`: (Never!).

---

## Go (Golang) Application

### Proof of Readability
This code demonstrates how easy it is to read a token's payload without the secret key.

```go
package main

import (
	"fmt"
	"github.com/golang-jwt/jwt/v5"
)

func InspectInsecureToken(tokenString string) {
	// 1. Define a map to hold the claims
	claims := jwt.MapClaims{}
	
	// 2. ParseUnverified decodes the token without checking the signature/secret
	token, _, err := new(jwt.Parser).ParseUnverified(tokenString, claims)
	
	if err != nil {
		fmt.Printf("Error parsing token: %v\n", err)
		return
	}

	// 3. Print the data - Proof that it is not encrypted
	fmt.Println("--- INSECURE PAYLOAD INSPECTION ---")
	if sub, ok := claims["sub"]; ok {
		fmt.Printf("Subject (Safe): %v\n", sub)
	}
	
	// If the token contained PII, we would see it here
	if email, ok := claims["email"]; ok {
		fmt.Printf("Email (UNSAFE!): %v\n", email)
	}
	
	fmt.Printf("Algorithm: %v\n", token.Method.Alg())
}
```

---

## Interview Questions

**Q1: What is the difference between Base64Url encoding and Encryption?**
**A:** Base64Url encoding is a reversible representation of binary data as text (ASCII). It provides zero security; anyone can decode it. Encryption transforms data using a key into ciphertext, which cannot be read without the corresponding decryption key.

**Q2: Why should you avoid putting an email address in a JWT?**
**A:** JWTs are visible to the client and any intermediary proxy. Placing PII like an email in the payload increases the risk of data leaks (violating GDPR) if the token is logged or intercepted. Use an opaque UUID (`sub`) instead.

**Q3: If you absolutely MUST transmit sensitive data in a token, what standard should you use?**
**A:** Use **JWE (JSON Web Encryption)**. JWE encrypts the payload content, ensuring that only the intended recipient (who holds the private key) can decrypt and read the information.

**Q4: Does signing a JWT (JWS) hide its contents?**
**A:** No. Signing only guarantees **Integrity** (detection of tampering). It does not provide **Confidentiality** (hiding the data).
