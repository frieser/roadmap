#API
---
---

## Summary
The Code Review Process is a critical component of the Secure Software Development Life Cycle (SSDLC) that combines human oversight with automated validation. It ensures that every code change is scrutinized for security vulnerabilities, logic flaws, and adherence to best practices before reaching production. By implementing a "Two-Person Rule," organizations mitigate the risk of malicious insider actions and accidental security regressions.

## Detailed Explanation

### The Two-Person Rule and Human Oversight
The **Two-Person Rule** (also known as the Four-Eyes Principle) dictates that at least two individuals must approve any change to the production codebase.
- **Insider Threat Mitigation**: No single developer can introduce malicious code (e.g., a backdoor) without collusion.
- **Knowledge Sharing**: Reviews act as a form of mentorship and ensure that knowledge of the system is distributed across the team.
- **Bias Reduction**: A second set of eyes often catches "blind spots" that the original author might have overlooked.

### Automated Linters (SAST) vs. Manual Review
A robust review process utilizes both automated tools and human intuition.

| Feature | Automated SAST (e.g., gosec) | Manual Review |
| :--- | :--- | :--- |
| **Speed** | Extremely fast (seconds/minutes) | Slow (hours/days) |
| **Scope** | Known patterns, syntax, hardcoded secrets | Complex logic, architecture, intent |
| **Reliability** | Consistent, never tires | Prone to human fatigue and oversight |
| **Context** | Limited (local file/package) | High (understands business requirements) |

**Integration Strategy**: Use SAST to filter out "low-hanging fruit" (simple vulnerabilities) so that human reviewers can focus on high-level security logic and complex attack vectors.

### Security Code Review Checklist
When performing a manual security review, focus on these critical areas:
1. **Injection Vulnerabilities**: Check if user-provided input is used in SQL queries, shell commands, or HTML templates without proper sanitization or parameterization.
2. **Sensitive Data Exposure**: Scan for hardcoded secrets (API keys, passwords) and ensure sensitive data is encrypted at rest and in transit.
3. **Authentication & Authorization**: Verify that every endpoint requires appropriate authentication and that users cannot access resources they don't own (IDOR).
4. **Error Handling**: Ensure errors don't leak stack traces or internal system information to the end-user.
5. **Logic Flaws**: Look for race conditions, integer overflows (in Go, check \`uint\` behavior), and "time-of-check to time-of-use" (TOCTOU) issues.

### Go (Golang) Context: golangci-lint and gosec
In the Go ecosystem, security automation is typically integrated into the CI/CD pipeline using \`golangci-lint\` and \`gosec\`.

#### Using gosec
\`gosec\` scans the Go AST to find common security issues.
\`\`\`bash
# Run gosec on the entire project
gosec ./...

# Output results in JSON for CI processing
gosec -fmt=json -out=results.json ./...
\`\`\`

#### Integrating with golangci-lint
\`golangci-lint\` is the industry standard for running multiple linters. It includes \`gosec\` as an optional linter.
\`.golangci.yml\` configuration:
\`\`\`yaml
linters:
  enable:
    - gosec
    - errcheck
    - staticcheck

linters-settings:
  gosec:
    # Exclude specific rules if necessary
    excludes:
      - G104 # Audit errors not checked
\`\`\`

#### Pre-commit Hook Example
Running security checks locally before pushing code:
\`\`\`yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/golangci/golangci-lint
    rev: v1.55.2
    hooks:
      - id: golangci-lint
\`\`\`

## Interview Questions

**Q: What is the primary benefit of the "Two-Person Rule" in a security context?**
**A:** It prevents any single individual from having absolute control over the code, mitigating the risk of malicious code injection (insider threat) and ensuring that at least one other person understands and validates the security implications of a change.

**Q: Why can't we rely solely on automated SAST tools for code reviews?**
**A:** SAST tools are excellent at finding pattern-based issues (like hardcoded secrets) but lack the context to understand business logic vulnerabilities, complex authorization flaws, or architectural weaknesses that a human reviewer can identify.

**Q: How do you handle "False Positives" in tools like gosec?**
**A:** False positives should be addressed by either fixing the code to be clearer or using "ignore" directives (like \`// #nosec\`) with a documented reason. Excessive false positives in CI should lead to tuning the tool's configuration to maintain a high signal-to-noise ratio.

**Q: What is the difference between SAST and DAST in a CI/CD pipeline?**
**A:** SAST (Static) analyzes the source code without executing it (finding vulnerabilities early), while DAST (Dynamic) tests the running application from the outside (finding runtime and configuration issues).
