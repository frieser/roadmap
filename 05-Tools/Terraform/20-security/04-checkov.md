---
tags: ['tools', 'roadmap']
---

# Checkov

## Summary
Checkov is an industry-leading static analysis tool for Infrastructure as Code (IaC). Developed by Bridgecrew (now part of Prisma Cloud by Palo Alto Networks), it scans cloud infrastructure configurations for security and compliance issues. It is highly regarded for its **graph-based analysis**, which allows it to understand complex relationships between resources rather than just checking individual attributes.

## Detailed Explanation

Checkov supports a wide range of formats, including Terraform, CloudFormation, Kubernetes, ARM Templates, and Serverless framework files.

### Key Features
*   **Graph-based Policies**: Can detect vulnerabilities that span multiple resources (e.g., a security group allowing public access to a specific EC2 instance).
*   **Software Composition Analysis (SCA)**: Scans for vulnerabilities in container images and open-source packages within the IaC project.
*   **Custom Policies**: Supports policies written in Python or YAML.
*   **Automated Remediations**: Often provides suggestions or even automated fixes for common misconfigurations.

### Implementation Examples

#### CLI Usage
Scanning a Terraform directory with specific checks:

```bash
# Scan a directory
checkov -d ./terraform

# Scan and skip specific checks
checkov -d ./terraform --skip-check CKV_AWS_20,CKV_AWS_57

# Output results in JSON for machine processing
checkov -d ./terraform -o json > results.json
```

#### Inline Suppression
Checkov allows developers to suppress specific warnings directly in the code, which is useful for documented exceptions.

```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-public-bucket"
  # checkov:skip=CKV_AWS_20: This bucket must be public for static website hosting
  acl    = "public-read"
}
```

#### CI/CD Integration (GitLab CI)
```yaml
checkov_scan:
  image: 
    name: bridgecrew/checkov:latest
    entrypoint: [""]
  script:
    - checkov -d . --soft-fail # soft-fail allows the pipeline to continue while still reporting errors
```

### Go Application
While Checkov is written in Python, it is a critical tool for **Go developers** building cloud-native applications. As Go is often used for writing the microservices that run on the infrastructure Checkov scans, ensuring the underlying infrastructure is secure is paramount for the overall security of the Go application.

## Interview Questions

**Q: What makes Checkov's "graph-based" scanning different from traditional linters?**
**A:** Traditional linters check resources in isolation. Checkov's graph-based approach builds a map of dependencies between resources, allowing it to identify risks that only emerge when resources are connected (e.g., an IAM role with excessive permissions attached to a public-facing instance).

**Q: How do you handle false positives in Checkov?**
**A:** False positives can be handled using **inline suppressions** (comments in the code) or by using the `--skip-check` flag in the CLI/CI configuration.

**Q: What is "Soft Fail" in Checkov?**
**A:** The `--soft-fail` flag allows the scan to run and report all findings without returning a non-zero exit code. This is useful during the initial integration of Checkov into a legacy project to avoid breaking existing pipelines while still gaining visibility.

**Q: Can Checkov scan container images?**
**A:** Yes, Checkov includes SCA capabilities that allow it to scan Dockerfiles and container images for known vulnerabilities (CVEs).
