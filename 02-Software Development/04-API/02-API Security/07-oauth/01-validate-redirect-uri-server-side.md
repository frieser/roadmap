#API #Security #OAuth
---
---

# Validate Redirect URI Server Side

## Summary
The `redirect_uri` is a critical security control in OAuth 2.0 flows. It dictates where the Authorization Server sends sensitive tokens or codes after authentication. Failure to strictly validate this URI enables **Open Redirect** attacks and **Token Leakage**, where attackers can steal authorization codes by tricking the server into redirecting the user to a malicious site.

## Detailed Explanation

### 1. The Role of Redirect URI
In the Authorization Code Flow:
1.  The User authenticates with the Provider (e.g., Google).
2.  The Provider redirects the User back to the Client Application using the `redirect_uri` provided in the initial request.
3.  This redirect contains the sensitive `code` (or `access_token` in implicit flows) in the query string or fragment.

### 2. Vulnerabilities
*   **Open Redirect**: If the server accepts any URI, an attacker can construct a link like `auth.com/login?redirect_uri=attacker.com`. The user sees a legitimate login page, but upon success, is sent to the attacker's site.
*   **Code Leakage**: If the attacker controls the redirection target, they capture the `code` from the URL parameters (e.g., `attacker.com/callback?code=SECRET_CODE`). They can then exchange this code for an Access Token (if PKCE is not enforced).

### 3. Best Practices (OAuth 2.1)
*   **Exact String Matching**: The server MUST perform a byte-for-byte comparison between the requested `redirect_uri` and the pre-registered URI.
*   **No Wildcards**:
    *   **Domain Wildcards** (`*.example.com`): Vulnerable to subdomain takeovers.
    *   **Path Wildcards** (`example.com/*`): Vulnerable if any page on `example.com` has an Open Redirect vulnerability.
*   **Absolute URIs**: Must include the scheme (`https`), domain, and path.
*   **No Fragments**: Fragments (`#`) are not sent to the server and should not be part of the registered URI.

---

## Go (Golang) Application

### Strict Validation Implementation
This example demonstrates exact matching logic, rejecting dangerous schemes and wildcards.

```go
package main

import (
	"fmt"
	"net/url"
	"strings"
)

// ValidateRedirectURI compares the requested URI against a whitelist of registered URIs.
func ValidateRedirectURI(requestedURI string, registeredURIs []string) (string, error) {
	// 1. Basic URL parsing to prevent malformed inputs
	parsed, err := url.Parse(requestedURI)
	if err != nil {
		return "", fmt.Errorf("invalid URI format")
	}

	// 2. Security Check: Must be Absolute
	if !parsed.IsAbs() {
		return "", fmt.Errorf("redirect_uri must be absolute")
	}

	// 3. Security Check: No Fragments allowed (RFC 6749)
	if parsed.Fragment != "" {
		return "", fmt.Errorf("redirect_uri must not contain fragments")
	}

	// 4. Security Check: Prevent dangerous schemes
	if strings.ToLower(parsed.Scheme) != "https" && strings.ToLower(parsed.Scheme) != "http" {
		// Note: http is only acceptable for localhost in dev
		return "", fmt.Errorf("scheme must be http(s)")
	}

	// 5. EXACT MATCHING (The Core Rule)
	for _, registered := range registeredURIs {
		if requestedURI == registered {
			return requestedURI, nil
		}
	}

	return "", fmt.Errorf("redirect_uri does not match any registered URI")
}
```

---

## Interview Questions

**Q1: Why are wildcard redirect URIs (e.g., `https://*.app.com`) considered insecure?**
**A:** Wildcards increase the attack surface. If an attacker can claim a subdomain (e.g., `evil.app.com`) or finds an Open Redirect vulnerability on any subdomain, they can steal the authorization code. Exact matching eliminates this risk.

**Q2: Can a `redirect_uri` contain a URL fragment (e.g., `#section1`)?**
**A:** No. RFC 6749 explicitly forbids fragments in the redirect URI sent to the authorization server. Fragments are client-side only and are not sent in HTTP requests. In Implicit flows, the server uses the fragment to append the token; pre-existing fragments could break this mechanism.

**Q3: How does validating the `redirect_uri` prevent Authorization Code Injection?**
**A:** It ensures the code is delivered *only* to the legitimate client application. Without validation, an attacker could redirect the code to their own server. While validation prevents the *delivery* to a wrong target, **PKCE** is the mechanism that prevents the *usage* of a stolen code.

**Q4: Is it safe to allow `http://localhost` as a redirect URI?**
**A:** Yes, but only for development or native apps (loopback). For production web applications, `https` is mandatory to prevent code interception over the network (Man-in-the-Middle).
