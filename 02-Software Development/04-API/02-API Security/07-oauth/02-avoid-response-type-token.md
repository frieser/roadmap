#API
---
---

# Avoid Response Type Token (Implicit Flow)

The **Implicit Grant Flow** (`response_type=token`) was originally designed for browser-based applications (SPAs) that could not securely store a client secret. However, modern security standards (OAuth 2.1) have deprecated it in favor of the **Authorization Code Flow with PKCE**.

## 1. Concept: Implicit Flow vs. Authorization Code Flow

| Feature | Implicit Flow (`token`) | Auth Code Flow (`code`) |
| :--- | :--- | :--- |
| **Token Delivery** | Directly in the URL fragment (`#`) | Exchanged server-side for a code |
| **Security** | Low (exposed in browser) | High (never touches the browser URL) |
| **Client Authentication** | Not required (Public Client) | Required (Confidential Client) or PKCE |
| **Refresh Tokens** | Not supported (typically) | Supported |

## 2. Vulnerability: Access Token Leakage

The primary reason to avoid `response_type=token` is that the access token is returned in the **URL fragment** (e.g., `https://app.com/callback#access_token=ya29...`).

### Risks
*   **Browser History**: URL fragments are often stored in browser history, allowing anyone with access to the device to retrieve the token.
*   **Referer Header**: If the page loads external resources (images, scripts) or links to other sites, the fragment (though technically not sent to servers by browsers) can sometimes be leaked via JavaScript or misconfigured proxies.
*   **JavaScript Access**: Any script running on the page (including malicious XSS) can access the token via `window.location.hash`.

## 3. Modern Standard: OAuth 2.1 & PKCE

**OAuth 2.1** and the **Security Best Current Practice (BCP)** strictly deprecate the Implicit Flow.

*   **PKCE (Proof Key for Code Exchange)**: Originally for mobile apps, it is now the standard for **all** OAuth clients, including SPAs.
*   **Recommendation**: Use `response_type=code` with PKCE. This ensures that even if the authorization code is intercepted, it cannot be exchanged for a token without the `code_verifier`.

## 4. Go (Golang) Examples

### Rejecting Implicit Flow on the Server
Authorization servers should explicitly reject requests using the implicit flow.

```go
func handleAuthorize(w http.ResponseWriter, r *http.Request) {
    responseType := r.URL.Query().Get("response_type")

    // BLOCK: Do not allow direct token issuance in URL
    if responseType == "token" {
        http.Error(w, "Implicit Flow is deprecated. Use response_type=code with PKCE.", http.StatusForbidden)
        return
    }

    // PROCEED: Handle Authorization Code Flow
    // ...
}
```

### Implementing PKCE Verification
When the client exchanges the `code` for a `token`, the server must verify the `code_verifier`.

```go
package oauth

import (
    "crypto/sha256"
    "encoding/base64"
    "fmt"
)

// ValidatePKCE verifies that the provided verifier matches the challenge using S256
func ValidatePKCE(codeVerifier, codeChallenge, method string) error {
    if method != "S256" {
        return fmt.Errorf("unsupported code_challenge_method: only S256 is allowed")
    }

    // S256: BASE64URL-ENCODE(SHA256(ASCII(code_verifier)))
    hash := sha256.Sum256([]byte(codeVerifier))
    computedChallenge := base64.RawURLEncoding.EncodeToString(hash[:])

    if computedChallenge != codeChallenge {
        return fmt.Errorf("invalid code_verifier: challenge mismatch")
    }

    return nil
}
```

## 5. Interview Preparation

### Questions & Answers

1.  **Q: Why is the Implicit Flow considered insecure for modern web apps?**
    *   **A:** Because the access token is exposed in the URL fragment, making it vulnerable to leakage through browser history, XSS, and `Referer` headers. It also lacks a secure mechanism for client authentication.

2.  **Q: What is the recommended alternative to Implicit Flow for SPAs?**
    *   **A:** Authorization Code Flow with PKCE (Proof Key for Code Exchange).

3.  **Q: How does PKCE prevent "Authorization Code Interception" attacks?**
    *   **A:** The client generates a secret `code_verifier` and sends its hash (`code_challenge`) during the authorization request. To exchange the code for a token, the client must provide the original `code_verifier`. An attacker who intercepts only the `code` cannot exchange it because they don't know the `code_verifier`.

4.  **Q: Does OAuth 2.1 allow the use of `response_type=token`?**
    *   **A:** No. OAuth 2.1 removes the Implicit Grant and Resource Owner Password Credentials Grant to improve the security baseline.
