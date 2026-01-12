---
tags: ['terraform', 'iac', 'tools', 'roadmap', 'jenkins', 'ci-cd']
---

# Terraform + Jenkins

## Summary
**Jenkins** is the most flexible automation server for Terraform, especially for on-premise or highly customized hybrid-cloud environments. Using **Declarative Pipelines (Groovy)**, Jenkins allows for complex logic, multi-stage approvals, and deep integration with custom internal tools.

## Detailed Explanation: Automation Pipelines

### Pipeline Structure (Groovy)
A typical Jenkins Terraform pipeline is defined in a `Jenkinsfile` and uses a Docker-based agent to ensure a consistent environment.

1.  **Agent Definition**: 
    *   Often uses `agent { docker { image 'hashicorp/terraform:latest' } }` to avoid version conflicts on the Jenkins controller or nodes.
2.  **Initialization**: 
    *   Runs `terraform init`. Credentials for the backend are often injected via the `withCredentials` block.
3.  **Planning**:
    *   Runs `terraform plan -out=tfplan`. 
    *   The `tfplan` binary is often archived as a build artifact to ensure it persists across the pipeline.
4.  **Manual Approval**:
    *   Uses the `input` step to pause execution. This sends a notification (e.g., via Slack or Email) to a human operator to review the plan and click "Proceed".
5.  **Application**:
    *   Runs `terraform apply tfplan` only after the manual approval is received.

### Security with Credentials Plugin
Jenkins handles secrets through the **Credentials Plugin**.
*   Secrets are stored in an encrypted database on the Jenkins controller.
*   The `withCredentials` wrapper masks these secrets in the console output, preventing accidental exposure of API keys.

### Shared Libraries
For organizations with many Terraform projects, common pipeline logic is often moved into a **Jenkins Shared Library**. This allows teams to use a single line (e.g., `terraformPipeline()`) to trigger a standardized, company-approved workflow.

## Go Context: Automation and Tooling
Jenkins is often used to run Go-based tools for infrastructure management, such as **Terratest** or custom CLI tools built with the **terraform-exec** library.

### Example: Running Terratest in Jenkins
```groovy
stage('Integration Tests') {
    steps {
        sh 'go test -v ./test'
    }
}
```

## Interview Questions

**Q: Why is it critical to use `terraform plan -out=tfplan` in a Jenkins pipeline?**
**A:** It ensures that the exact configuration that was reviewed and approved in the "Plan" stage is the one executed in the "Apply" stage. Without this, the infrastructure could potentially change between the two steps, leading to unexpected results.

**Q: How do you handle secrets securely in a Jenkins Terraform pipeline?**
**A:** By using the **Credentials Plugin** and the `withCredentials` block. This allows you to inject secrets into environment variables for the duration of a script block while ensuring they are masked in the build logs.

**Q: What is the benefit of using Docker agents for Terraform in Jenkins?**
**A:** It provides environment isolation and version consistency. Each job can use a specific version of Terraform (via a Docker tag) without requiring that version to be installed globally on the Jenkins nodes, preventing "version hell."

**Q: How can you implement a manual "gate" in a Jenkins Declarative Pipeline?**
**A:** By using the `input` step. This pauses the pipeline and can be configured to require specific user permissions to approve, ensuring that only authorized personnel can trigger a production deployment.
