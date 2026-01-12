---
tags: ['tools', 'roadmap']
---

# Terrascan

## Summary
Terrascan is an open-source static code analyzer for Infrastructure as Code (IaC) that helps identify security risks and compliance violations. Developed by Accurics (now part of Tenable), it uses the **Open Policy Agent (OPA)** engine and **Rego** query language to evaluate code across various platforms including Terraform, Kubernetes, Helm, and Docker.

## Detailed Explanation

Terrascan's primary goal is to "Shift Left" security by identifying misconfigurations early in the development lifecycle, before they reach production.

### Key Features
*   **Multi-IaC Support**: Works with Terraform, Kubernetes, ArgoCD, CloudFormation, and more.
*   **500+ Built-in Policies**: Comes with a large library of policies covering best practices for AWS, Azure, GCP, and Kubernetes.
*   **Extensibility**: Users can write custom policies using Rego (the standard OPA language).
*   **Drift Detection**: Can detect changes in the runtime environment that diverge from the defined IaC.

### Implementation Examples

#### CLI Usage
To scan a directory containing Terraform files:

```bash
# Initialize terrascan (downloads latest policies)
terrascan init

# Scan a directory
terrascan scan -i terraform -d ./infrastructure
```

#### CI/CD Integration (GitHub Actions)
Terrascan can be easily integrated into a CI pipeline to block builds that contain critical security flaws.

```yaml
name: Terrascan Scan
on: [push, pull_request]

jobs:
  terrascan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Run Terrascan
        uses: tenable/terrascan-action@main
        with:
          iac_type: 'terraform'
          iac_dir: './'
          severity: 'high'
          only_warn: false # Fail the build on high severity findings
```

### OPA and Rego
Since Terrascan uses OPA, a policy for ensuring S3 buckets are not public might look like this in Rego:

```rego
package terrascan

deny[msg] {
    resource := input.aws_s3_bucket[name]
    resource.acl == "public-read"
    msg := sprintf("S3 bucket '%v' is publicly readable", [name])
}
```

### Go Application
Terrascan is written in **Go**. For Go developers, this means the tool is highly performant and can be extended by contributing to its core codebase or by using its libraries to build custom security scanners.

## Interview Questions

**Q: What engine does Terrascan use for policy evaluation?**
**A:** Terrascan uses the **Open Policy Agent (OPA)** engine and the **Rego** language to define and execute its security policies.

**Q: How does Terrascan support "Shift Left" security?**
**A:** By allowing developers to run scans locally on their machines or within CI/CD pipelines, it identifies vulnerabilities in the code *before* any resources are actually provisioned in the cloud.

**Q: Can Terrascan scan resources already deployed in the cloud?**
**A:** Yes, Terrascan has features for scanning runtime environments to detect "drift" between the deployed infrastructure and the original IaC definition.

**Q: What is the advantage of using Rego for policies in Terrascan?**
**A:** Rego is a vendor-neutral, industry-standard language for policy-as-code. Using it allows teams to share policies between different tools (like Terrascan for IaC and OPA Gatekeeper for Kubernetes).
