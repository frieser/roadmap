#API #Security #OAuth
---
---

# Default and Validated Scopes

## Summary
Scopes in OAuth 2.0 define the **permissions** (access rights) a client is requesting for a resource. Implementing **Validated Scopes** ensures that clients cannot request more access than they are allowed ("Scope Creep"). The server must validate requested scopes against a whitelist and enforce the **Principle of Least Privilege** by assigning a restricted set of **Default Scopes** when none are requested.

## Detailed Explanation

### 1. Principle of Least Privilege
OAuth scopes allow for granular access control. Instead of granting "full access" to a user's account, a client might only need `read:profile` or `write:posts`.
*   **Security Benefit**: If the client's Access Token is stolen, the damage is limited to the scopes granted to that token.

### 2. The Risk: Scope Creep
*   **Malicious Clients**: An attacker might modify the authorization URL to request `scope=admin` or `scope=*`. If the server blindly grants whatever is requested, security is compromised.
*   **Lazy Developers**: Developers often request broad scopes ("just in case") rather than what is strictly needed.

### 3. Best Practices
1.  **Server-Side Validation**: The Authorization Server (AS) must check every requested scope against the client's registered "Allowed Scopes."
2.  **Intersection Logic**: `Granted Scopes = Requested Scopes ∩ Allowed Scopes`.
3.  **Default Scopes**: If the `scope` parameter is missing, default to the *minimum* possible access (e.g., just `openid`), not full access.
4.  **Notify on Change**: If the AS grants fewer scopes than requested (due to policy), it should inform the client, although often the AS simply ignores unknown scopes (RFC 6749 Section 3.3).

---

## Go (Golang) Application

### Validating and Intersection Logic
Efficiently calculating the granted scopes using Go maps (Sets).

```go
package main

import (
	"fmt"
	"strings"
)

// CalculateGrantedScopes determines the final list of scopes based on request and permissions.
func CalculateGrantedScopes(requested []string, allowedClientScopes []string) []string {
	// 1. Create a set (map) of allowed scopes for O(1) lookups
	allowedSet := make(map[string]bool)
	for _, s := range allowedClientScopes {
		allowedSet[s] = true
	}

	var granted []string

	// 2. Default Scope Logic
	if len(requested) == 0 {
		// If no scope requested, return a safe default (e.g., read-only public info)
		// Or return the intersection of "default" and "allowed"
		if allowedSet["read:public"] {
			return []string{"read:public"}
		}
		return []string{}
	}

	// 3. Intersection Logic: Only grant what is both Requested AND Allowed
	for _, req := range requested {
		if allowedSet[req] {
			granted = append(granted, req)
		} else {
			// Optional: Log that a restricted scope was requested and denied
			fmt.Printf("Warning: Client requested unauthorized scope '%s'\n", req)
		}
	}

	return granted
}

func main() {
	allowed := []string{"read:profile", "read:posts", "write:posts"}
	requested := []string{"read:profile", "delete:users", "admin:all"}

	finalScopes := CalculateGrantedScopes(requested, allowed)
	
	// Output: Granted: [read:profile]
	// "delete:users" and "admin:all" are silently dropped or cause an error depending on policy
	fmt.Printf("Granted Scopes: %v\n", finalScopes)
}
```

---

## Interview Questions

**Q1: What is the purpose of the `scope` parameter in OAuth 2.0?**
**A:** It allows the client to specify the level of access (permissions) it needs. This enables the user to consent to specific actions (e.g., "Allow this app to read your emails but not send them") and implements the Principle of Least Privilege.

**Q2: What should the Authorization Server do if a client requests a scope it is not authorized for?**
**A:** According to RFC 6749, the server has two options:
1.  **Error**: Reject the request with `invalid_scope`.
2.  **Partial Grant**: Ignore the unauthorized scope and grant only the valid ones. If the server grants fewer scopes than requested, it shouldn't treat it as an error but the client must be written to handle the reduced access. Best practice usually favors strict validation (Error) to avoid client confusion.

**Q3: Why is it dangerous to have a default scope of "all" or "full_access"?**
**A:** If a developer forgets to include the `scope` parameter (or if an attacker omits it intentionally), the token will be issued with full administrative privileges. Defaults should always be the most restrictive option possible (e.g., read-only public profile) to minimize impact in case of errors.

**Q4: How does Scope Validation protect against malicious clients?**
**A:** It prevents "Privilege Escalation." Even if a malicious client (or a compromised legitimate client) requests `admin` access, the server's validation logic ensures they only receive the scopes they were explicitly registered/approved for during onboarding.
