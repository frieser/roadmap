---
tags: ['tools', 'roadmap']
---

# KICS (Keeping Infrastructure as Code Secure)

## Summary
KICS (Keeping Infrastructure as Code Secure) is an open-source static analysis tool developed by Checkmarx. It is designed to find security vulnerabilities, compliance issues, and infrastructure misconfigurations in various IaC formats. KICS is particularly strong in its **multi-format support** and extensive library of over 2,000 queries.

## Detailed Explanation

KICS provides a robust engine that parses IaC files into a common internal representation, which is then queried using Rego logic.

### Key Features
*   **Broad Coverage**: Supports Terraform, Ansible, CloudFormation, Kubernetes, Helm, Dockerfiles, and more.
*   **2000+ Queries**: One of the largest open-source databases of IaC security checks.
*   **Extensible Architecture**: Built on top of the Open Policy Agent (OPA) engine.
*   **Multiple Output Formats**: Supports JSON, SARIF, HTML, and PDF reports, making it easy to integrate with vulnerability management platforms.

### Implementation Examples

#### CLI Usage
Scanning a project and generating a SARIF report (ideal for GitHub Code Scanning):

```bash
# Scan a directory
kics scan -p ./terraform -o ./results

# Specify output format
kics scan -p . --report-formats sarif --output-path ./reports
```

#### Docker Usage
Running KICS without installing it locally:

```bash
docker run -v $PWD:/path checkmarx/kics:latest scan -p /path -o /path/results.json
```

#### CI/CD Integration (GitHub Actions)
```yaml
name: KICS Scan
on: [push]
jobs:
  kics-job:
    runs-on: ubuntu-latest
    name: KICS Scan
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: KICS Github Action
        uses: checkmarx/kics-action@v1.7
        with:
          path: 'terraform'
          output_path: 'results/'
```

### Go Application
KICS is written in **Go**. Its ability to handle massive IaC projects with thousands of files is a testament to the concurrency model and performance of the Go language. For developers, KICS provides a CLI that is familiar and fast, fitting perfectly into a DevOps toolchain.

## Interview Questions

**Q: How many queries does KICS provide out of the box?**
**A:** KICS provides over **2,000 built-in queries** covering a wide range of platforms and security standards like CIS Benchmarks.

**Q: Which language is used to write KICS queries?**
**A:** KICS uses **Rego**, the query language for the Open Policy Agent (OPA).

**Q: What is a SARIF report, and why is it useful in KICS?**
**A:** SARIF (Static Analysis Results Interchange Format) is a standard format for static analysis tools. KICS can export results in SARIF, which allows them to be natively displayed in platforms like GitHub's "Security" tab.

**Q: Does KICS support configuration management tools like Ansible?**
**A:** Yes, one of KICS's strengths is its support for both infrastructure provisioning (Terraform) and configuration management (Ansible) tools.
