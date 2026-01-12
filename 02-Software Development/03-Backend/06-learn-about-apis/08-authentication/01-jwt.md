---
---

## Summary
JSON Web Token (JWT) is an open standard (RFC 7519) for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs are commonly used for authorization and information exchange.

## Detailed Explanation
JWTs are "stateless" tokens. Once issued, the server doesn't need to look up a session in a database; it only needs to verify the signature.

### Structure of a JWT
A JWT consists of three parts separated by dots:
1. **Header**: Specifies the algorithm used (e.g., HS256) and the type of token.
2. **Payload**: Contains the claims (e.g., user ID, roles, expiration time).
3. **Signature**: Used to verify that the sender is who they say they are and that the message wasn't changed.

### Workflow
1. Client logs in with credentials.
2. Server verifies credentials and issues a signed JWT.
3. Client stores the JWT (e.g., in localStorage or an HTTP-only cookie).
4. For every subsequent request, the client sends the JWT in the `Authorization: Bearer <token>` header.
5. Server verifies the signature and expiration, then grants access based on the payload.

## Go Context
In Go, `golang-jwt/jwt` is the standard library for working with JWTs.

### Example: Creating and Verifying a JWT
```go
package main

import (
	"fmt"
	"time"
	"github.com/golang-jwt/jwt/v5"
)

var mySigningKey = []byte("secret-key")

func CreateToken(userID string) (string, error) {
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
		"user_id": userID,
		"exp":     time.Now().Add(time.Hour * 24).Unix(),
	})
	return token.SignedString(mySigningKey)
}

func main() {
	token, _ := CreateToken("123")
	fmt.Println("Token:", token)
}
```

## Interview Questions
- **Q: What is the difference between HS256 and RS256?**
- **A:** HS256 (HMAC with SHA-256) is a symmetric algorithm using one secret key for both signing and verification. RS256 (RSA with SHA-256) is an asymmetric algorithm using a private key to sign and a public key to verify.

- **Q: Where should you store a JWT on the client side?**
- **A:** Storing it in an `HttpOnly` and `Secure` cookie is generally more secure than `localStorage` because it protects against Cross-Site Scripting (XSS).

- **Q: Can you revoke a JWT?**
- **A:** Because JWTs are stateless, you cannot "log out" a user in the traditional sense. To revoke them, you either need a short expiration time or implement a "blacklist" or "revocation list" (which makes the API somewhat stateful).
