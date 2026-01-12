#API #Security #OAuth
---
---

# Use State Parameter (CSRF Protection)

## Summary
The `state` parameter is a standard OAuth 2.0 security mechanism designed to prevent **Cross-Site Request Forgery (CSRF)**, specifically **Login CSRF**. It acts as a cryptographically secure "nonce" that binds the user's browser session to the authorization request. If the `state` returned by the provider does not match the one stored in the user's session, the request is rejected.

## Detailed Explanation

### 1. The Attack: Login CSRF
In a Login CSRF attack, the attacker wants to force the victim to log in to the *attacker's* account.
1.  **Attacker** starts an OAuth flow and gets an authorization code (but doesn't use it).
2.  **Attacker** tricks the **Victim** into clicking a link that feeds this code to the victim's client application.
3.  **Victim's Client** uses the code to log the victim in.
4.  **Result**: The victim is now logged in as the attacker. Any private data the victim uploads (photos, documents) is now saved to the attacker's account.

### 2. The Solution: Binding Session to Request
The `state` parameter ensures that the user who *started* the flow is the same one *finishing* it.

1.  **Initiation**: Client generates a random string `state = "xyz123"`, stores it in the user's session (cookie), and redirects the user to `auth-server.com?state=xyz123`.
2.  **Callback**: The Auth Server redirects back to `client.com/callback?state=xyz123`.
3.  **Verification**: The Client checks if `cookie.state == query.state`.

### 3. Best Practices
*   **High Entropy**: Use a cryptographically secure random number generator (CSPRNG).
*   **Session Bound**: The state must be tied to the specific browser session (e.g., via an HttpOnly cookie).
*   **Single Use**: Ideally, verify and clear the state immediately.

---

## Go (Golang) Application

### Generating and Verifying State
Using `crypto/rand` for secure generation and cookies for session binding.

```go
package main

import (
	"crypto/rand"
	"encoding/base64"
	"fmt"
	"net/http"
)

// GenerateState creates a 32-byte random string
func GenerateState() (string, error) {
	b := make([]byte, 32)
	_, err := rand.Read(b) // Use CSPRNG
	if err != nil {
		return "", err
	}
	return base64.URLEncoding.EncodeToString(b), nil
}

func LoginHandler(w http.ResponseWriter, r *http.Request) {
	state, _ := GenerateState()

	// 1. Store state in a secure, HttpOnly cookie
	http.SetCookie(w, &http.Cookie{
		Name:     "oauth_state",
		Value:    state,
		Path:     "/",
		HttpOnly: true,
		Secure:   true, // Mandatory in production
		SameSite: http.SameSiteLaxMode,
		MaxAge:   300, // 5 minutes expiration
	})

	// 2. Redirect with state
	redirectURL := fmt.Sprintf("https://provider.com/auth?state=%s", state)
	http.Redirect(w, r, redirectURL, http.StatusTemporaryRedirect)
}

func CallbackHandler(w http.ResponseWriter, r *http.Request) {
	// 1. Get state from URL
	queryState := r.URL.Query().Get("state")

	// 2. Get state from Cookie
	cookie, err := r.Cookie("oauth_state")
	if err != nil {
		http.Error(w, "State cookie missing", http.StatusBadRequest)
		return
	}

	// 3. VERIFY: They must match exactly
	if queryState != cookie.Value {
		http.Error(w, "State mismatch! Potential CSRF attack.", http.StatusForbidden)
		return
	}

	// 4. Cleanup: Delete the cookie to prevent reuse
	http.SetCookie(w, &http.Cookie{Name: "oauth_state", MaxAge: -1})

	fmt.Fprintf(w, "State verified. Proceeding with token exchange...")
}
```

---

## Interview Questions

**Q1: What specific attack does the OAuth `state` parameter prevent?**
**A:** It prevents **Login CSRF** (Cross-Site Request Forgery). This is where an attacker tricks a victim into logging into the attacker's account, potentially leading to the victim uploading sensitive data to an account the attacker controls.

**Q2: Can I use a timestamp as the `state` parameter?**
**A:** No. A timestamp is predictable. An attacker could guess the timestamp and forge a valid state. The state must be a **Cryptographically Secure Pseudo-Random Number (CSPRNG)** to be unguessable.

**Q3: Does PKCE replace the need for the `state` parameter?**
**A:** For preventing **CSRF**, mostly yes. Since the PKCE `code_verifier` is stored in the session and required to exchange the code, an attacker cannot inject a code because they don't have the victim's `code_verifier`. However, `state` is still recommended by RFCs as a standard mechanism specifically for CSRF protection and restoring application state (e.g., redirecting the user back to the page they were on).

**Q4: How should the state be stored on the client side?**
**A:** It should be stored in a way that is bound to the user's session but inaccessible to JavaScript (to prevent XSS theft). An **HttpOnly, Secure, SameSite Cookie** is the best storage mechanism for web applications.
