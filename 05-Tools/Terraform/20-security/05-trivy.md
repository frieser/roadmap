---
tags: ['tools', 'roadmap']
---

# Trivy

## Summary
Trivy is a comprehensive, multi-purpose security scanner developed by Aqua Security. While it gained fame as a container image scanner, it has evolved into a "universal" tool capable of scanning **Infrastructure as Code (IaC)**, file systems, Git repositories, and even virtual machine images. It is known for its incredible speed, accuracy, and ease of use.

## Detailed Explanation

Trivy's IaC scanning engine supports Terraform, CloudFormation, Dockerfiles, and Kubernetes manifests.

### Key Features
*   **Universal Scanner**: One tool for containers, IaC, secrets, and software licenses.
*   **Misconfiguration Detection**: Identifies insecure settings in cloud templates.
*   **Vulnerability Scanning (CVEs)**: Detects known vulnerabilities in OS packages and application dependencies.
*   **Secret Scanning**: Finds hardcoded passwords, tokens, and API keys.
*   **Extensible Policies**: Uses Rego (the OPA language) for its misconfiguration checks.

### Implementation Examples

#### CLI Usage
Scanning a Terraform project for misconfigurations:

```bash
# Scan a directory for IaC misconfigurations
trivy iac ./infrastructure

# Scan for secrets in the current directory
trivy fs --scanners secret .

# Combine scans (Vulnerabilities, Secrets, IaC)
trivy fs --scanners vuln,secret,config .
```

#### CI/CD Integration (GitHub Actions)
```yaml
name: Trivy Scan
on: [push]
jobs:
  build:
    name: Trivy Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Run Trivy vulnerability scanner in IaC mode
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'config'
          hide-progress: false
          format: 'table'
          exit-code: '1' # Fail the build
          severity: 'CRITICAL,HIGH'
```

### Go Application
Trivy is written in **Go**. Its performance and "single binary" nature are hallmarks of Go's efficiency. For Go developers, Trivy is often the tool of choice because it integrates seamlessly into Go-based development workflows and is often used to scan the very Docker images that package Go binaries.

```bash
# Example: Scanning a Go project's local directory
trivy fs .
```

## Interview Questions

**Q: Why is Trivy often preferred in CI/CD pipelines over other scanners?**
**A:** Trivy is extremely fast and comes as a single binary with no dependencies, making it very easy to install and run in ephemeral CI environments.

**Q: What categories of security risks can Trivy detect?**
**A:** Trivy can detect **Vulnerabilities** (CVEs), **Misconfigurations** in IaC, **Secrets** (leaked credentials), and **License** compliance issues.

**Q: How does Trivy stay up to date with the latest vulnerabilities?**
**A:** Trivy automatically downloads a vulnerability database (cached locally) every time it runs, ensuring it has the latest CVE information without requiring manual updates.

**Q: Can you use custom policies with Trivy?**
**A:** Yes, Trivy uses the OPA engine, allowing users to write and include custom policies using the **Rego** language.
