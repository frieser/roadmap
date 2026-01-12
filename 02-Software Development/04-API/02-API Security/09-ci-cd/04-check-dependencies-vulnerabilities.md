#API
---
---

## Summary
Dependency Vulnerability Scanning (often called SCA - Software Composition Analysis) is the process of identifying known security vulnerabilities in the open-source libraries and third-party modules your project relies on. Since modern applications often consist of 80%+ third-party code, this is a critical attack vector.

## Detailed Explanation

### How it Works
1.  **Manifest Parsing**: The scanner reads package manager files (`go.mod`, `package-lock.json`, `pom.xml`) to list all dependencies and their specific versions.
2.  **Database Lookup**: It checks these versions against a vulnerability database (like NVD - National Vulnerability Database, or GitHub Advisory Database).
3.  **Reporting**: It reports any CVEs (Common Vulnerabilities and Exposures) found, their severity, and usually the "fixed in" version.

### Transitive Dependencies
The tool must traverse the entire dependency tree. You might use `Lib A`, which is safe, but `Lib A` uses `Lib B` which has a critical vulnerability. Your app is still vulnerable.

### Mitigation
*   **Patch**: Upgrade to a safe version.
*   **Workaround**: If no patch exists, remove the dependency or mitigate the specific vulnerable function usage.
*   **Accept Risk**: If the vulnerable code path isn't used.

## Go-Specific Context/Examples

Go has excellent tooling for this, integrated directly into the toolchain.

### Tools
1.  **govulncheck**: The official Go vulnerability scanner. It is superior to generic scanners because it performs **call graph analysis**. It only flags a vulnerability if your code *actually calls* the vulnerable function, reducing false positives.
    ```bash
    go install golang.org/x/vuln/cmd/govulncheck@latest
    govulncheck ./...
    ```
2.  **Nancy**: A tool that checks `go.sum` against Sonatype's OSS Index.

### Example: `go.mod` security
Ideally, you should pin dependencies using `go.sum` (checksums) to prevent "Supply Chain Attacks" where a malicious actor changes the code of a published version.

## Interview Questions

**Q: What is a CVE?**
**A:** CVE stands for **Common Vulnerabilities and Exposures**. It is a list of publicly disclosed cybersecurity vulnerabilities, each having a unique ID (e.g., CVE-2021-44228 for Log4Shell) and a CVSS score indicating severity (0-10).

**Q: Why is `govulncheck` better than just checking `go.mod` versions?**
**A:** Checking versions only tells you if you *have* a bad library. `govulncheck` parses your code to see if you *use* the vulnerable part of that library. If you import a library but don't call the vulnerable function, `govulncheck` won't block you, whereas generic scanners would.

**Q: What is the risk of using "latest" or range-based versions (e.g., `^1.0.0`) in dependencies?**
**A:** It makes builds non-deterministic. If a library releases a broken or malicious patch, your next build will automatically pull it in, potentially breaking prod or introducing a backdoor. Always use lock files (`go.sum`, `package-lock.json`) to ensure every build uses the exact same code.
