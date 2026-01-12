# GitHub REST API

## Summary
The GitHub REST API allows you to interact with GitHub programmatically. You can create repos, manage issues, and trigger actions using standard HTTP requests.

## Detailed Explanation

### Base URL
`https://api.github.com`

### Authentication
*   **Personal Access Token (PAT)**: Used for scripts.
*   **OAuth Token**: Used for apps.
*   Header: `Authorization: Bearer <token>`

### Usage
```bash
curl -H "Authorization: Bearer $TOKEN" https://api.github.com/user/repos
```

### Go-specific Context
The standard library `google/go-github` is the official client.
```go
import "github.com/google/go-github/github"
client := github.NewClient(nil)
```

## Interview Questions
**Q: What is the rate limit for unauthenticated requests?**
**A:** 60 requests per hour. Authenticated is 5,000 per hour.

**Q: How does pagination work?**
**A:** Via the `Link` header in the response.
