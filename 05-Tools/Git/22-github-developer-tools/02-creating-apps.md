# Creating GitHub Apps

## Summary
To create an integration, you register it in Developer Settings. You define the permissions, webhooks, and homepage URL.

## Detailed Explanation

### Steps
1.  Profile > Settings > Developer settings.
2.  New GitHub App.
3.  **Webhook URL**: Where GitHub sends payloads (your Go server).
4.  **Permissions**: Select minimal privileges.
5.  **Private Key**: Generate a PEM file to sign JWTs for authentication.

### Go-specific Context
To build a GitHub App in Go:
1.  Use `bradleyfalzon/ghinstallation` to handle the JWT authentication dance.
2.  Listen for webhooks using `net/http`.
3.  Parse payloads with `go-github`.

## Interview Questions
**Q: What is the "Client Secret"?**
**A:** A secret string used to verify the identity of the application during the OAuth flow. Never share it.
