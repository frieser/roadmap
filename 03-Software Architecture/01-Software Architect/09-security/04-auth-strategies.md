---
---
#security #architecture
# Authentication Strategies

## Summary
Authentication (AuthN) is the process of verifying who a user is. For a Software Architect, choosing an authentication strategy involves balancing security, scalability, user experience, and complexity. Modern systems favor **Token-based Auth (JWT)** for stateless scalability and **OAuth 2.0/OIDC** for delegated authorization and identity management. **Session Management** remains relevant for stateful web applications where revocation is critical. **MFA** and **SSO** are essential for enterprise-grade security and user convenience.

---

## Detailed Development

### 1. Basic Auth vs Token-based Auth (JWT)
*   **Basic Authentication**:
    *   **Mechanism**: Sends `username:password` encoded in Base64 in the `Authorization` header (`Basic <credentials>`).
    *   **Pros**: Simple to implement, no special client logic.
    *   **Cons**: Credentials sent with every request; requires HTTPS; hard to implement MFA; logout is difficult (requires browser cache clearing).
*   **Token-based Auth (JWT)**:
    *   **Mechanism**: Server issues a signed JSON Web Token (JWT) after initial login. Client sends `Bearer <token>` in headers.
    *   **Pros**: Stateless (no server-side session lookup), cross-domain friendly, scalable.
    *   **Cons**: Tokens are "stale" once issued (hard to revoke); size can grow with claims; sensitive to secret/key compromise.

### 2. OAuth 2.0 and OIDC (OpenID Connect)
*   **OAuth 2.0**: A framework for **Authorization**, not Authentication. It allows a third party to access resources on behalf of a user.
    *   **Authorization Code Flow**: Used for web/mobile apps. Best practice: Use with **PKCE** (Proof Key for Code Exchange) to prevent code injection.
    *   **Client Credentials Flow**: Used for Machine-to-Machine (M2M) communication (e.g., microservice A talking to microservice B).
*   **OIDC**: An identity layer on top of OAuth 2.0. It adds an `ID Token` (JWT) containing user profile information, standardizing the **Authentication** part.

### 3. Session Management (Stateful vs Stateless)
*   **Stateful (Session-based)**:
    *   Server creates a session record (DB/Redis) and sends a `Session ID` via a cookie (`HttpOnly`, `Secure`).
    *   **Best for**: Banking, high-security apps where instant session termination (logout/revocation) is required.
*   **Stateless (Token-based)**:
    *   The state is carried in the token itself (JWT).
    *   **Best for**: Microservices, high-traffic APIs, mobile apps.
    *   **Revocation Strategy**: Use short-lived Access Tokens + long-lived Refresh Tokens (stored in DB/Redis for revocation).

### 4. MFA and SSO
*   **Multi-Factor Authentication (MFA)**:
    *   Combines Knowledge (Password), Possession (TOTP, Hardware Key), or Inherence (Biometrics).
    *   **TOTP**: Time-based One-Time Password (e.g., Google Authenticator).
*   **Single Sign-On (SSO)**:
    *   **SAML 2.0**: XML-based, traditional in corporate environments.
    *   **OIDC**: JSON/JWT based, preferred for modern web/mobile apps.
    *   **Architectural Benefit**: Centralizes identity management, reduces attack surface, and improves DX/UX.

---

## Go & TypeScript Examples

### JWT Verification (Go)
```go
func VerifyToken(tokenString string, secret []byte) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(token *jwt.Token) (interface{}, error) {
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
        }
        return secret, nil
    })
    if err != nil || !token.Valid {
        return nil, fmt.Errorf("invalid token")
    }
    return token.Claims.(*Claims), nil
}
```

### OAuth2 PKCE Flow (TypeScript/Deno)
```typescript
const url = new URL("https://auth.provider.com/authorize")
url.searchParams.set("client_id", CLIENT_ID)
url.searchParams.set("response_type", "code")
url.searchParams.set("code_challenge", codeChallenge) // Generated via SHA256
url.searchParams.set("code_challenge_method", "S256")
url.searchParams.set("redirect_uri", CALLBACK_URL)
```

---

## Go-Specific Applications
In Go, the `golang.org/x/oauth2` package is the standard for implementing OAuth2 clients. For JWT, `github.com/golang-jwt/jwt` is widely used. Middleware like `Chi` or `Echo` often provide built-in JWT/Auth wrappers. Architects should prefer `OIDC` discovery endpoints to automatically fetch public keys (JWKS) for token validation.

---

## Interview Preparation Questions
1. **Explain the difference between OAuth 2.0 and OIDC.** (OAuth is for access/authorization, OIDC is for identity/authentication).
2. **Why is PKCE recommended for mobile/SPA apps instead of the standard Auth Code flow?** (To protect the code from being intercepted on public clients where a client secret cannot be stored securely).
3. **How do you revoke a JWT?** (Blacklisting/Denylisting tokens in Redis until expiry, or using Refresh Tokens to issue new Access Tokens).
4. **When would you choose stateful sessions over stateless JWTs?** (When immediate session invalidation is a strict requirement, or when the payload is too large for a token).
