#API #Security #JWT
---
---

# Short Token Expiration (TTL)

## Summary
Short Token Expiration (TTL) is a security best practice for **JSON Web Tokens (JWT)** that minimizes the window of opportunity for an attacker if a token is compromised. Because JWTs are stateless and difficult to revoke before they expire, keeping their lifetime short (e.g., 5-15 minutes) ensures that stolen tokens become useless quickly. To maintain a good user experience, this is almost always paired with a **Refresh Token** strategy.

## Detailed Explanation

### 1. Access Token vs. Refresh Token
Modern authentication systems use two types of tokens to balance security and usability:

*   **Access Token**: 
    -   **Purpose**: Sent in the `Authorization: Bearer <token>` header to access protected resources.
    -   **Lifetime**: Very short (e.g., 5 - 15 minutes).
    -   **Nature**: Stateless (verified by signature, no DB lookup needed).
*   **Refresh Token**:
    -   **Purpose**: Used only to request a new Access Token once the current one expires.
    -   **Lifetime**: Long (e.g., 7 days, 30 days, or "until revoked").
    -   **Nature**: Statefull (often stored in a database/Redis) to allow for immediate revocation.

### 2. Security Trade-offs
*   **Revokability Problem**: JWTs are "fire and forget." Once issued, the server cannot easily "kill" them without implementing a blacklist (which reintroduces state).
*   **The Mitigation**: A short TTL limits the damage. If an attacker steals an access token via XSS, they only have access for ~10 minutes.
*   **Refresh Token Rotation**: A technique where every time a Refresh Token is used, a *new* Refresh Token is issued, and the old one is invalidated. This helps detect theft (if the old token is reused, it signals a breach, and the system can lock the account).

### 3. Recommended TTL Values
| Token Type | High Security (Banking) | Standard Web App |
| :--- | :--- | :--- |
| **Access Token** | 2 - 5 minutes | 15 - 30 minutes |
| **Refresh Token** | 1 - 12 hours | 7 - 30 days |

---

## Go (Golang) Application

### Setting Expiration (exp)
In Go, using the `golang-jwt/jwt` library, we set the `exp` claim as a Unix timestamp.

```go
package main

import (
	"time"
	"github.com/golang-jwt/jwt/v5"
)

var secretKey = []byte("your-very-secure-secret")

func GenerateAccessToken(userID string) (string, error) {
	claims := jwt.MapClaims{
		"sub": userID,
		"iat": time.Now().Unix(),
		// Short TTL: 15 minutes
		"exp": time.Now().Add(time.Minute * 15).Unix(),
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString(secretKey)
}
```

### Refresh Flow Pattern
This is a simplified handler demonstrating how a refresh token is exchanged for a new access token.

```go
func RefreshHandler(refreshToken string) (string, error) {
	// 1. Validate the refresh token (verify signature AND check DB for revocation)
	if IsRevoked(refreshToken) {
		return "", fmt.Errorf("token revoked")
	}

	// 2. Extract UserID from the refresh token
	userID := GetUserID(refreshToken)

	// 3. Generate a new short-lived access token
	newAccessToken, err := GenerateAccessToken(userID)
	if err != nil {
		return "", err
	}

	// 4. Optional: Rotate Refresh Token here
	
	return newAccessToken, nil
}
```

---

## Interview Questions

**Q1: Why do we use short TTLs for JWTs instead of long ones?**
**A:** Since JWTs are stateless, they cannot be easily revoked by the server. A short TTL limits the "window of abuse" if a token is stolen via XSS or man-in-the-middle attacks.

**Q2: How do you handle a user "logging out" if the JWT is still valid?**
**A:** You delete the token from the client. On the server, since you can't technically expire the JWT, you must either wait for it to expire (short TTL helps here) or implement a "Deny List" (Blacklist) in Redis with a TTL matching the JWT's expiration.

**Q3: What is Refresh Token Rotation and why use it?**
**A:** It is a security technique where every use of a refresh token invalidates it and issues a new one. If an attacker steals a refresh token, they race the legitimate user to use it. Once used, the token is dead. If the legitimate user tries to use the old token later (and fails), it signals to the server that the token was stolen/reused, triggering a security alert or account lockout.

**Q4: Where is the safest place to store a Refresh Token in a browser?**
**A:** In an **HttpOnly, Secure, and SameSite=Strict cookie**. This prevents client-side JavaScript (XSS) from reading the token, unlike `localStorage`.
