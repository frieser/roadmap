---
---

## Summary
SAML (Security Assertion Markup Language) is an open standard for exchanging authentication and authorization data between parties, specifically between an Identity Provider (IdP) and a Service Provider (SP). It is an XML-based protocol primarily used in enterprise Single Sign-On (SSO) scenarios.

## Detailed Explanation
SAML is older than OIDC and is widely used in corporate environments where users log in once to their company network (IdP) and gain access to many different applications (SPs) like Slack, Zoom, or AWS.

### Key Components
- **Identity Provider (IdP)**: The system that authenticates the user (e.g., Okta, Microsoft Azure AD).
- **Service Provider (SP)**: The application the user wants to access.
- **Assertion**: An XML document containing information about the user and their permissions, signed by the IdP.

### Workflow (SP-Initiated)
1. User tries to access the Service Provider.
2. SP redirects the user to the IdP with a SAML Request.
3. IdP authenticates the user.
4. IdP sends the user back to the SP's Assertion Consumer Service (ACS) URL with a SAML Assertion.
5. SP validates the XML signature and grants access.

## Go Context
Working with SAML in Go usually involves specialized libraries like `crewjam/saml`.

### Example: SAML Middleware setup
```go
import "github.com/crewjam/saml/samlsp"

// setup SAML middleware
middleware, _ := samlsp.New(samlsp.Options{
    URL:            *baseURL,
    Key:            key,
    Certificate:    certificate,
    IDPMetadataURL: idpMetadataURL,
})
// use middleware.RequireAccount to protect routes
```

## Interview Questions
- **Q: When would you use SAML instead of OpenID Connect?**
- **A:** SAML is the standard for enterprise SSO and is required by most large corporate IT departments. OIDC is more modern, lightweight (JSON vs XML), and preferred for consumer-facing apps and mobile devices.

- **Q: What is "XML Signature Wrapping" (XSW) in SAML?**
- **A:** It is a common security vulnerability where an attacker modifies the XML structure to inject their own assertions while keeping a valid signature on a different part of the document. Modern libraries are designed to prevent this.

- **Q: What is a SAML Metadata file?**
- **A:** It is an XML file shared between the IdP and SP that contains configuration info, such as endpoint URLs and the public keys used for signing and encryption.
