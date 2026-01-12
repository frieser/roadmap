#API #Security #JWT #Performance
---
---

# Keep Payload Small

## Summary
The **"Keep Payload Small"** principle is crucial for JWT performance and reliability. Unlike session cookies which are tiny references, JWTs are "By-Value" tokens—they carry all their data with them. Every byte added to the payload increases the size of **every** API request, consuming bandwidth, increasing latency, and potentially hitting HTTP header size limits in web servers (Nginx/Apache).

## Detailed Explanation

### 1. Network Bandwidth & Latency
*   **Per-Request Overhead**: A JWT is typically sent in the `Authorization` header. If a user makes 50 requests, the JWT is sent 50 times.
*   **Bloat Multiplier**: JWTs are Base64Url encoded, which adds ~33% overhead to the raw JSON size. A 1KB JSON payload becomes ~1.3KB on the wire.
*   **Mobile Impact**: On high-latency mobile networks (3G/4G), large headers can exceed the **MTU (Maximum Transmission Unit)** of ~1500 bytes, causing TCP packet fragmentation. This requires extra round-trips to reassemble the packet, noticeably slowing down the request.

### 2. HTTP Server Limits (Silent Failures)
Web servers and CDNs have strict limits on HTTP header sizes. If a JWT exceeds this limit, the request is dropped with a generic `4xx` error before it even reaches your application code.

| Component | Typical Limit | Consequence |
| :--- | :--- | :--- |
| **Nginx** | 4KB - 8KB | `414 Request-URI Too Large` or `400 Bad Request` |
| **Apache** | 8KB | `400 Bad Request` |
| **Node.js** | 8KB - 16KB | `431 Request Header Fields Too Large` |
| **Cloudflare** | 16KB | Request dropped at edge |

### 3. Best Practices
*   **Minimal Claims**: Store only the `sub` (User ID), `exp`, and perhaps a current `role` or `scope`.
*   **Reference, Don't Embed**: Instead of embedding a list of 50 permissions (`"can_edit_posts", "can_delete_users"...`), embed a `role_id` or `group_id` and look up the permissions on the server (using Redis/Memcached for speed).
*   **Short Keys**: Use shortened keys in the JSON payload (e.g., `"u"` instead of `"username"`, `"r"` instead of `"roles"`) if extreme optimization is needed (common in IoT).

---

## Go (Golang) Application

### Efficient Struct Mapping
Demonstrating how to map a struct to a standard set of claims without bloating the token.

```go
package main

import (
	"time"
	"github.com/golang-jwt/jwt/v5"
)

// CustomClaims - Keeping it lean
type CustomClaims struct {
	UserID string `json:"sub"`   // Standard claim
	Role   string `json:"role"`  // Just the high-level role
	
	// Embedding standard claims for exp, iat, etc.
	jwt.RegisteredClaims
}

func CreateLeanToken(userID, role string) (string, error) {
	// AVOID: Embedding large arrays or user profile data
	// Bad Example: ProfileImageURL, Bio, LastLoginIP, [Array of 50 Permissions]
	
	claims := CustomClaims{
		UserID: userID,
		Role:   role,
		RegisteredClaims: jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(time.Now().Add(time.Hour)),
			IssuedAt:  jwt.NewNumericDate(time.Now()),
			Issuer:    "my-app",
		},
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString([]byte("secret"))
}
```

---

## Interview Questions

**Q1: How does a large JWT affect the performance of a mobile application?**
**A:** It increases the overhead of every API request. Crucially, if the headers exceed the TCP MTU (~1500 bytes), the packet is fragmented, causing additional round-trips. On high-latency mobile networks, this can significantly delay the "Time to First Byte" (TTFB).

**Q2: You are receiving randomized `400 Bad Request` errors on your API, but only for users with many permissions. What is the likely cause?**
**A:** The JWT has likely grown too large (due to the permissions array) and is exceeding the HTTP Header size limit of the load balancer (e.g., Nginx default 4KB) or the web server. The fix is to remove permissions from the token and use a `role_id` or `group_id` instead.

**Q3: What is the downside of using "Reference Tokens" (Server-side lookup) vs "Value Tokens" (Self-contained JWT)?**
**A:** Reference tokens require a database or cache lookup on every request to validate the token and fetch user data, which adds latency and database load. JWTs (Value Tokens) avoid this lookup but suffer from the bandwidth/size issues discussed above. A hybrid approach (JWT + Redis for invalidation/permissions) is often the sweet spot.
