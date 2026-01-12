# Security Best Practices

## Summary
Security Best Practices in engineering management involve embedding security into the entire Software Development Life Cycle (SDLC), rather than treating it as an afterthought. This includes secure coding standards, automated vulnerability scanning, access control management, and cultivating a "security-first" culture among developers.

## Detailed Explanation
Security is a shared responsibility. As a manager, you are responsible for the security posture of your team's software. This is achieved through a combination of process, tooling, and education.

### Key Pillars
1.  **Shift Left**: Integrating security testing (SAST/DAST) early in the development process (e.g., in the IDE or PR checks) rather than waiting for a pre-release audit.
2.  **Principle of Least Privilege (PoLP)**: Ensuring services and users have only the permissions necessary to perform their function.
3.  **Dependency Management**: Regularly scanning and updating third-party libraries (Supply Chain Security) to avoid vulnerabilities like Log4Shell.
4.  **Defense in Depth**: Layering security controls (WAF, authentication, authorization, encryption at rest/transit).
5.  **OWASP Top 10**: Ensuring the team understands and mitigates common web vulnerabilities (Injection, Broken Auth, XSS, etc.).

### Managerial Responsibilities
- **Threat Modeling**: conducting design reviews where attackers' perspectives are considered ("What could go wrong?").
- **Security Training**: Providing regular training for engineers.
- **Incident Response Plan**: Having a clear runbook for when (not if) a breach occurs.

## Go Code Example
This example demonstrates a secure configuration manager that sanitizes inputs and handles secrets securely, preventing common pitfalls like logging credentials or hardcoding secrets. It also models a basic Role-Based Access Control (RBAC) check.

```go
package main

import (
	"errors"
	"fmt"
	"log"
	"os"
)

// SecurityContext simulates a request context with user identity
type SecurityContext struct {
	UserID string
	Role   string
}

// ConfigLoader handles loading configuration securely
type ConfigLoader struct {
	sensitiveKeys map[string]bool
}

func NewConfigLoader() *ConfigLoader {
	return &ConfigLoader{
		sensitiveKeys: map[string]bool{
			"DB_PASSWORD": true,
			"API_KEY":     true,
			"JWT_SECRET":  true,
		},
	}
}

// LoadEnvVar safely loads environment variables without leaking secrets in logs
func (c *ConfigLoader) LoadEnvVar(key string) (string, error) {
	val := os.Getenv(key)
	if val == "" {
		return "", fmt.Errorf("missing required environment variable: %s", key)
	}

	// Log that we loaded it, but REDACT the value if it's sensitive
	logValue := val
	if c.sensitiveKeys[key] {
		logValue = "[REDACTED]"
	}
	fmt.Printf("Loaded Config: %s=%s\n", key, logValue)

	return val, nil
}

// AccessControlService enforces RBAC
type AccessControlService struct {
	// Map of Resource -> Minimum Role Required
	policies map[string]string
}

// roleHierarchy defines role power: Admin > User > Guest
var roleHierarchy = map[string]int{
	"guest": 0,
	"user":  1,
	"admin": 2,
}

func (acs *AccessControlService) Authorize(ctx SecurityContext, resource string) error {
	requiredRole, ok := acs.policies[resource]
	if !ok {
		// Secure by default: if no policy exists, deny access
		return errors.New("access denied: no policy defined for resource")
	}

	userLevel := roleHierarchy[ctx.Role]
	reqLevel := roleHierarchy[requiredRole]

	if userLevel < reqLevel {
		return fmt.Errorf("access denied: user role '%s' insufficient for resource requiring '%s'", ctx.Role, requiredRole)
	}

	fmt.Printf("Access GRANTED for user %s to resource %s\n", ctx.UserID, resource)
	return nil
}

func main() {
	// 1. Secure Configuration Loading
	loader := NewConfigLoader()
	
	// Simulating env vars
	os.Setenv("DB_PASSWORD", "superSecret123")
	os.Setenv("APP_MODE", "production")

	// This is safe to log
	_, _ = loader.LoadEnvVar("APP_MODE")
	// This will be redacted in logs
	_, _ = loader.LoadEnvVar("DB_PASSWORD")

	// 2. Authorization Check
	authService := &AccessControlService{
		policies: map[string]string{
			"/api/admin/users": "admin",
			"/api/profile":     "user",
			"/api/public":      "guest",
		},
	}

	// Scenario: Regular user trying to access admin resource
	ctx := SecurityContext{UserID: "alice", Role: "user"}
	
	err := authService.Authorize(ctx, "/api/admin/users")
	if err != nil {
		fmt.Printf("Security Alert: %v\n", err)
	}

	// Scenario: Regular user accessing profile
	_ = authService.Authorize(ctx, "/api/profile")
}
```

## Interview Questions
1.  **How do you ensure your team keeps third-party dependencies secure?**
    *   *Focus*: Automated scanning (Dependabot, Snyk), policy on updates, minimizing dependencies.
2.  **Explain "Shift Left" in the context of security. How have you implemented it?**
    *   *Focus*: Catching issues early, SAST/DAST integration, developer responsibility.
3.  **What is your approach to handling secrets (API keys, passwords) in an application?**
    *   *Focus*: Never in git, environment variables, secret management systems (Vault, AWS Secrets Manager).
4.  **You discover a critical vulnerability in production code. Walk me through your immediate response.**
    *   *Focus*: Incident response process, triage, patching, communication, post-mortem.
