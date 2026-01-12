---
---

## Summary
The OWASP Top 10 is a standard awareness document for developers and web application security. It represents a broad consensus on the most critical security risks to web applications.

## Detailed Explanation
The list is updated every few years. As of the most recent consensus (2025 perspective):

### Top Risks
1. **Broken Access Control**: Users can access data or functions outside of their permissions.
2. **Cryptographic Failures**: Sensitive data (like passwords) is not properly encrypted or hashed.
3. **Software Supply Chain Failures**: Vulnerabilities in third-party libraries or build pipelines.
4. **Injection**: Unfiltered user input is executed as code (SQL Injection, XSS).
5. **Insecure Design**: Flaws in the architecture rather than just the implementation.
6. **Security Misconfiguration**: Default passwords left unchanged, unnecessary ports open, etc.
7. **Authentication Failures**: Weak passwords, lack of MFA, or flawed session management.
8. **Software and Data Integrity Failures**: Updates or data received from untrusted sources without verification.
9. **Logging and Monitoring Failures**: Not recording security events, allowing attackers to remain undetected.
10. **Server-Side Request Forgery (SSRF)**: Tricking a server into making requests to internal or external systems it shouldn't access.

## Go Context
Go's strong typing and standard library help prevent many of these.

### Example: Preventing SQL Injection
```go
// BAD: Concatenating strings
db.Query("SELECT * FROM users WHERE id = " + userID)

// GOOD: Using parameterized queries
db.Query("SELECT * FROM users WHERE id = ?", userID)
```

## Interview Questions
- **Q: What is SQL Injection?**
- **A:** It is a vulnerability where an attacker injects malicious SQL code into a query via user input. This can allow them to read, modify, or delete database data. It is prevented by using parameterized queries (Prepared Statements).

- **Q: How do you prevent Broken Access Control?**
- **A:** Implement a "Deny by Default" policy. Every request should verify that the authenticated user has the specific permission required for that resource or action.

- **Q: What is SSRF?**
- **A:** Server-Side Request Forgery occurs when an attacker can control the URL that a server-side application makes a request to. This can be used to scan internal networks or access metadata services in cloud environments.
