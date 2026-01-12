---
---

# Default and Validated Scopes

## Summary
In OAuth 2.0, **scopes** are the mechanism used to limit an application's access to a user's account. Proper validation and defaulting of these scopes are critical to maintaining the **Principle of Least Privilege (PoLP)** and preventing "Scope Creep."

## Detailed Development

### 1. Concept: Scope and Least Privilege
OAuth 2.0 scopes are essentially "tags" representing specific permissions.
*   **Granularity**: Scopes should be as specific as possible (e.g., `user:email` instead of `user`).
*   **PoLP**: An application should only request the minimum set of scopes required to function. This minimizes the "blast radius" if the application's access token is leaked.

### 2. Risk: Scope Creep
**Scope Creep** occurs when:
*   **Broad Scopes**: Developers use overly permissive scopes (e.g., `cloud_platform` instead of `storage:read`) to simplify development.
*   **Unnecessary Requests**: Clients request scopes "just in case" they might need them later.
*   **Impact**: Increases the risk of unauthorized data access and makes user consent screens misleading or overwhelming.

### 3. Best Practices for the Authorization Server (AS)

*   **Mandatory Validation**: The AS must verify that the requested scopes are valid for the specific `client_id` and the `resource_owner`.
*   **Intersection Logic**: The final token should contain: `GrantedScopes = RequestedScopes ∩ ClientAllowedScopes`.
*   **Handling Unknown Scopes**:
    *   **Fail**: Reject the request with an `invalid_scope` error.
    *   **Ignore**: Process the request but omit the unknown scope (must notify the client in the token response if the granted scope differs from the requested one).
*   **Default Scopes**: If a client sends an authorization request without a `scope` parameter, the AS should assign a pre-defined, highly restricted "default scope" (e.g., `openid` or `profile:read`).
*   **Server-Side Enforcement**: The Resource Server (RS) must always check the scopes in the Access Token before serving data.

---

## Go (Golang) Implementation

### Scope Validation & Intersection Logic
In Go, we often use `map[string]bool` to perform efficient set operations on scopes.

```go
package main

import (
	"fmt"
	"strings"
)

// ValidateScopes performs intersection between requested and allowed scopes.
// It returns the granted scopes and a boolean indicating if any requested scope was invalid.
func ValidateScopes(requested string, allowed []string) ([]string, bool) {
	if requested == "" {
		// Return default scope if none requested
		return []string{"read:public"}, true
	}

	allowedMap := make(map[string]bool)
	for _, s := range allowed {
		allowedMap[s] = true
	}

	requestedSlice := strings.Fields(requested)
	var granted []string
	allValid := true

	for _, s := range requestedSlice {
		if allowedMap[s] {
			granted = append(granted, s)
		} else {
			allValid = false
		}
	}

	return granted, allValid
}

func main() {
	allowed := []string{"user:email", "repo:read", "profile"}
	requested := "user:email repo:write"

	granted, allValid := ValidateScopes(requested, allowed)

	fmt.Printf("Requested: %s\n", requested)
	fmt.Printf("Granted:   %v\n", granted)   // Output: [user:email]
	fmt.Printf("All Valid: %t\n", allValid) // Output: false
}
```

---

## Go-Specific Applications
When using the `golang.org/x/oauth2` library, the `Config` struct holds the requested scopes:

```go
conf := &oauth2.Config{
    ClientID:     "YOUR_CLIENT_ID",
    ClientSecret: "YOUR_CLIENT_SECRET",
    Scopes:       []string{"SCOPE1", "SCOPE2"}, // These are the requested scopes
    Endpoint:     github.Endpoint,
}
```

On the **server-side** (if building an OAuth2 provider), frameworks like [Fosite](https://github.com/ory/fosite) handle the complexity of scope validation and intersection automatically.

---

## Interview Preparation

### Questions & Answers

**Q: What is the risk of having a "wildcard" scope?**
**A:** A wildcard scope (like `*`) bypasses the Principle of Least Privilege. If a client with this scope is compromised, the attacker has full access to every API resource the user owns, which could lead to catastrophic data loss.

**Q: How should an Authorization Server respond if a client requests an unauthorized scope?**
**A:** According to RFC 6749, the server can either return an `invalid_scope` error or issue the token with the reduced (authorized) scopes. If it reduces the scopes, it **must** include the `scope` parameter in the token response to inform the client of the change.

**Q: Why is it important to assign a default scope if the client provides none?**
**A:** If no default is assigned, the behavior might be unpredictable (some servers might grant "all" scopes by default, which is a massive security risk). Assigning a minimal default ensures that the application operates under the most restrictive permissions by default.

**Q: What is "Scope Creep" and how does it affect user trust?**
**A:** Scope creep is the gradual expansion of requested permissions beyond what is necessary. It degrades user trust because users may feel the application is "spying" on them if it asks for access to contacts or location when it only needs to read a profile name.
