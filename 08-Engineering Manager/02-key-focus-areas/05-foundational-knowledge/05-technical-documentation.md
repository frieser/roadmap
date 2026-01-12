## Summary
Technical documentation is the information that describes software to its users, administrators, and developers. It includes API references, architecture decision records (ADRs), setup guides, and inline code comments. Good documentation acts as a force multiplier, enabling asynchronous work and faster onboarding.

## Detailed Explanation
### Types of Documentation
*   **Code Documentation**: Comments explaining *why* complex logic exists.
*   **API Documentation**: Contracts for usage (Swagger/OpenAPI).
*   **System Architecture**: High-level diagrams and flowcharts (MermaidJS).
*   **Operational**: Runbooks, disaster recovery procedures.

### Best Practices
*   **Docs-as-Code**: Treat documentation like code (version controlled, reviewed).
*   **Keep it fresh**: Outdated documentation is worse than no documentation.
*   **Know your audience**: Don't explain basic syntax to senior devs; don't use internal jargon for end-users.

### Go Code Example: Idiomatic Go Documentation
Go has a built-in documentation tool, `godoc`. Writing good documentation in Go involves writing clear comments directly above declarations.

```go
// Package auth provides authentication services for the platform.
// It supports JWT issuance and validation.
package auth

import (
	"fmt"
	"time"
)

// User represents an authenticated entity in the system.
type User struct {
	ID       string // Unique UUID
	Username string
	Role     string // "admin" or "user"
}

// TokenGenerator defines the behavior for creating session tokens.
type TokenGenerator interface {
	// GenerateToken creates a signed string token for a given user
	// that expires after the specified duration.
	// Returns an error if signing fails.
	GenerateToken(u User, expiry time.Duration) (string, error)
}

// JWTService implements TokenGenerator using HS256.
type JWTService struct {
	secretKey []byte
}

// NewJWTService creates a new service instance.
// Panics if secretKey is empty.
func NewJWTService(secretKey string) *JWTService {
	if secretKey == "" {
		panic("auth: secret key cannot be empty")
	}
	return &JWTService{secretKey: []byte(secretKey)}
}

// Deprecated: Use GenerateToken instead.
func (s *JWTService) CreateToken(u User) string {
	return "old-token"
}

func main() {
	// This code is self-documenting via the comments above.
	// Running 'go doc' on this package would generate a formatted manual.
	fmt.Println("Documentation example")
}
```

## Interview Questions
**Q: How do you ensure documentation stays up to date?**
**A:** I advocate for a "Definition of Done" that includes documentation updates. If a feature changes the API, the PR cannot be merged until the Swagger spec is updated. I also encourage "Docs-as-Code" where documentation lives in the same repo as the code, making it easier to update atomically.

**Q: What is the difference between code comments and external documentation?**
**A:** Code comments (inline) should explain the *why* and the *nuance* of specific implementation details that aren't obvious from reading the code itself. External documentation (READMEs, Wikis) should explain the *what* and *how*—how to install, architecture overviews, and how modules interact.

**Q: What are Architecture Decision Records (ADRs)?**
**A:** ADRs are short text files that capture an important architectural decision, the context, the options considered, and the consequences. They are crucial for preserving institutional memory so future team members understand *why* we chose PostgreSQL over Mongo, for example.
