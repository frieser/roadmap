---
---

## Summary
OAuth 2.0 is an industry-standard protocol for authorization. It doesn't focus on *who* the user is, but rather *what* they are allowed to do. It allows applications to obtain limited access to user accounts on an HTTP service (like Google, Facebook, or GitHub).

## Detailed Explanation
OAuth is about "delegated authority." For example, allowing a website to access your Google contacts without giving that website your Google password.

### Key Roles
- **Resource Owner**: The user who owns the data.
- **Client**: The application requesting access.
- **Resource Server**: The server holding the data (e.g., Google APIs).
- **Authorization Server**: The server issuing tokens (e.g., Google accounts).

### Common Grant Types
- **Authorization Code**: Most common for web apps. Involves a temporary code exchanged for an access token.
- **Client Credentials**: For machine-to-machine communication.
- **Implicit**: (Deprecated) Formerly for SPAs.
- **Password**: (Deprecated) Exchange user credentials directly for a token.

## Go Context
Go provides the `golang.org/x/oauth2` package to handle OAuth flows.

### Example: Google OAuth2 Config
```go
package main

import (
	"golang.org/x/oauth2"
	"golang.org/x/oauth2/google"
)

var googleOauthConfig = &oauth2.Config{
	ClientID:     "YOUR_CLIENT_ID",
	ClientSecret: "YOUR_CLIENT_SECRET",
	RedirectURL:  "http://localhost:8080/callback",
	Scopes:       []string{"https://www.googleapis.com/auth/userinfo.email"},
	Endpoint:     google.Endpoint,
}
```

## Interview Questions
- **Q: What is the difference between Authentication and Authorization?**
- **A:** Authentication is proving *who* you are (Identity). Authorization is proving *what* you can do (Permissions). OAuth is primarily an authorization framework.

- **Q: What is a Redirect URI?**
- **A:** It is a URL in the client application where the authorization server sends the user (and the authorization code) after the user grants access. It must be pre-registered to prevent security attacks.

- **Q: What is the "state" parameter used for in OAuth?**
- **A:** It is a unique string sent by the client and returned by the authorization server. It is used to prevent Cross-Site Request Forgery (CSRF) by ensuring that the response matches the request initiated by the user.
