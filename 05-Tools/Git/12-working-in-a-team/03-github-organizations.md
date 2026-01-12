# GitHub Organizations

## Summary
An Organization is a shared account type for businesses and open-source projects. It allows you to manage many repositories and users under one roof, with centralized security and billing.

## Detailed Explanation

### Features
*   **Centralized Access Control**: Manage who sees what.
*   **Teams**: Group users (Engineering, Sales).
*   **SAML SSO**: Enterprise feature to login via Okta/Google.
*   **Audit Logs**: See who did what.

### Go-specific Context
Most Go libraries live under an organization (e.g., `github.com/gin-gonic`, `github.com/spf13`) rather than a personal user, to ensure continuity if the original author leaves.

## Interview Questions
**Q: Can an organization fork a repository?**
**A:** Yes.

**Q: Can I transfer my personal repo to an organization?**
**A:** Yes, via Settings > Transfer repository.
