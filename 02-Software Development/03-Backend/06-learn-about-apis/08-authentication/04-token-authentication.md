---
---

## Summary
Token-based authentication is a broad term for systems where a server issues a unique token to a client after successful login. The client sends this token with every subsequent request. This is the foundation for modern web and mobile apps.

## Detailed Explanation
Token authentication replaces the traditional "Session ID in a cookie" approach.

### Types of Tokens
1. **Opaque Tokens**: Random strings stored in a database on the server. The server must look up the token for every request.
2. **Self-Contained Tokens (JWT)**: Contain the user information and signature inside the token itself. The server doesn't need a database lookup.

### Workflow
1. Client sends username/password.
2. Server validates and generates a token.
3. Server returns the token to the client.
4. Client stores the token and sends it in the `Authorization: Bearer <token>` header.

## Go Context
Go developers often build custom middleware to validate tokens.

### Example: Simple Token Middleware
```go
func TokenMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		token := r.Header.Get("X-Auth-Token")
		if token != "valid-token-123" { // In reality, check DB or JWT
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}
		next.ServeHTTP(w, r)
	})
}
```

## Interview Questions
- **Q: What is a "Bearer Token"?**
- **A:** A bearer token is a security token that gives access to the "bearer" of the token. No other proof of identity is required. If someone steals your bearer token, they can act as you.

- **Q: Why use tokens instead of session cookies?**
- **A:** Tokens are better for mobile apps (which don't handle cookies as naturally as browsers), they are naturally cross-domain (easier for CORS), and self-contained tokens (JWT) allow for better horizontal scaling.

- **Q: How do you handle token expiration?**
- **A:** You typically use an "Access Token" with a short life (e.g., 15 minutes) and a "Refresh Token" with a long life (e.g., 7 days). When the access token expires, the client uses the refresh token to get a new access token.
