#API
---
---

## Summary
Continuous Security Analysis involves integrating automated security testing tools into the CI/CD pipeline. By scanning code and applications at every commit or deployment, teams can identify vulnerabilities early (Shift Left) rather than waiting for a penetration test or a breach. It primarily consists of SAST (Static Application Security Testing) and DAST (Dynamic Application Security Testing).

## Detailed Explanation

### Methodologies
1.  **SAST (Static Application Security Testing)**:
    *   **What**: Analyzes source code, byte code, or binaries for security vulnerabilities without executing the application.
    *   **When**: During the build/commit phase.
    *   **Pros**: Finds vulnerabilities early, exact file/line location.
    *   **Cons**: High false positives, cannot find runtime issues (config errors).
2.  **DAST (Dynamic Application Security Testing)**:
    *   **What**: Analyzes the running application from the outside, like a hacker would.
    *   **When**: After deployment to a staging environment.
    *   **Pros**: Finds real runtime issues (auth bypass, headers).
    *   **Cons**: Slower, requires a running app, less detail on where in code the fix is.
3.  **IAST (Interactive)**: Hybrid approach using agents inside the app to monitor execution in real-time.

### CI/CD Integration
*   **Pipeline Gate**: Fail the build if critical severity vulnerabilities are found.
*   **Reporting**: Export results to dashboard (SonarQube, DefectDojo).

## Go-Specific Context/Examples

In the Go ecosystem, **gosec** is the standard tool for SAST.

### Example: Running `gosec` in CI

1.  **Install gosec**:
    ```bash
    go install github.com/securego/gosec/v2/cmd/gosec@latest
    ```
2.  **Run Scan**:
    ```bash
    # Scan all files, exclude test files, fail on medium/high severity
    gosec -exclude-dir=test -severity medium ./...
    ```

### Common Go Vulnerabilities Detected
*   **G101**: Hardcoded credentials.
*   **G104**: Errors not handled (audit).
*   **G201**: SQL query construction using `fmt.Sprintf` (SQL Injection).
*   **G401/G501**: Usage of weak cryptographic hashes (MD5, SHA1).

## Interview Questions

**Q: Why is "Shift Left" important in security?**
**A:** "Shift Left" means moving security testing to the earliest possible stage in the development lifecycle (the "left" side of the timeline). Fixing a bug during coding costs ~$25; fixing it in production can cost $10,000+ plus reputational damage.

**Q: What is the difference between SAST and DAST?**
**A:** SAST inspects the *code* (white-box) and runs early (build time). DAST inspects the *running application* (black-box) and runs later (staging/test time). Effective security requires both.

**Q: How do you handle False Positives in automated security scans?**
**A:** You should tune the ruleset to ignore irrelevant checks. For valid code that triggers a warning (e.g., using MD5 for non-crypto hashing), use a suppression comment (e.g., `// #nosec G401`) to explicitly mark it as reviewed and safe, rather than disabling the check globally.
