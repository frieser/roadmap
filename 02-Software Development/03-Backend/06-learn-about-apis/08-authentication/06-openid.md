---
---

## Summary
OpenID Connect (OIDC) is an identity layer built on top of the OAuth 2.0 protocol. While OAuth 2.0 is about authorization (accessing data), OIDC is about authentication (verifying who the user is).

## Detailed Explanation
OIDC allows clients to verify the identity of an end-user based on authentication performed by an Authorization Server.

### The ID Token
In addition to the Access Token (for data), OIDC introduces the **ID Token**. This is a JWT that contains information about the authenticated user (like name, email, and subject ID).

### Workflow
1. Client redirects user to OIDC provider (e.g., Google).
2. User logs in.
3. Provider sends an Authorization Code back to the client.
4. Client exchanges the code for an Access Token AND an ID Token.
5. Client decodes the ID Token to know who the user is.

## Go Context
The `coreos/go-oidc` package is the standard for implementing OIDC in Go.

### Example: Initializing an OIDC Provider
```go
import "github.com/coreos/go-oidc/v3/oidc"

provider, _ := oidc.NewProvider(ctx, "https://accounts.google.com")
verifier := provider.Verifier(&oidc.Config{ClientID: "YOUR_CLIENT_ID"})
```

## Interview Questions
- **Q: What is the difference between OAuth 2.0 and OpenID Connect?**
- **A:** OAuth 2.0 is for authorization (getting a key to a house), while OIDC is for authentication (showing an ID card to prove who you are). OIDC uses OAuth 2.0 as its foundation.

- **Q: What is the `sub` claim in an ID Token?**
- **A:** "sub" stands for Subject. It is a unique, non-reassignable identifier for the user within the provider's system. You should use this as the primary key for the user in your database.

- **Q: What is the Discovery Endpoint in OIDC?**
- **A:** It is a standard URL (usually `/.well-known/openid-configuration`) where a provider publishes its capabilities, supported scopes, and the URLs for its authorization and token endpoints.
