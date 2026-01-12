---
tags: ['tools', 'roadmap']
---

## Summary
**Infracost** is an open-source tool that provides **cloud cost estimates** for Terraform directly in the development workflow (Shift Left). By parsing HCL code or plan files, Infracost allows engineers to see the financial impact of infrastructure changes on Pull Requests before the resources are actually provisioned.

## Detailed Explanation

### 1. What is Infracost?
Infracost sits between code and cloud. Unlike traditional billing tools that look at past spending, Infracost estimates future costs. It supports major cloud providers including **AWS, Azure, and Google Cloud**.

### 2. Core Features
- **Cost Diffs on PRs:** Automatically comments on Pull Requests with a breakdown of cost changes.
- **Guardrails/Policy:** Flag or block PRs that exceed a certain budget or percentage increase.
- **CLI Tool:** Run breakdowns and diffs locally during development.
- **Integration:** Plugs into GitHub Actions, GitLab CI, Azure DevOps, and more.

### 3. Usage Commands

#### Breakdown
Shows the total monthly cost of every resource in the project.
```bash
infracost breakdown --path .
```

#### Diff
Shows the change in monthly cost between the current state and proposed changes.
```bash
infracost diff --path . --compare-to baseline.json
```

#### Using Plan Files (Most Accurate)
For complex projects with variables, it is better to use a Terraform plan file.
```bash
terraform plan -out tfplan.binary
terraform show -json tfplan.binary > plan.json
infracost breakdown --path plan.json
```

### 4. CI/CD Workflow
A typical Infracost integration in a pipeline:
1. **Checkout:** Fetch the code.
2. **Infracost Setup:** Install the CLI and configure API keys.
3. **Generate Diff:** Compare the current branch against the main branch.
4. **Post Comment:** Send the result back to the PR interface for visibility.

### 5. Example Output
```text
~ aws_instance.web_server
  +$30.37 ($0.00 -> $30.37)
    + t3.medium (730 hours)
```
- **~**: Resource modified or added.
- **+$30.37**: The estimated monthly increase.
- **t3.medium**: The specific cost component (instance type).

## Interview Questions

**Q: What is "Shift Left" in the context of cloud costs?**
**A:** It means moving cost awareness and management earlier in the development lifecycle (to the "left"). Instead of waiting for a monthly bill, developers see cost estimates during the coding and PR review stage.

**Q: How does Infracost calculate prices?**
**A:** Infracost uses its own Cloud Pricing API, which mirrors the public pricing pages of AWS, Azure, and GCP. It maps Terraform resource attributes (like instance size or storage volume) to the corresponding price points.

**Q: Why would you use Infracost instead of just checking the AWS Cost Explorer?**
**A:** AWS Cost Explorer is reactive (shows what happened). Infracost is proactive (shows what will happen). Infracost integrates with the developer's Git workflow, allowing for cost-based peer reviews before any money is spent.

**Q: Can Infracost block a deployment if it's too expensive?**
**A:** Yes. By integrating Infracost into a CI/CD pipeline, you can set "Guardrails" that cause the CI build to fail if a change exceeds a specific budget or a percentage increase, preventing expensive mistakes.
