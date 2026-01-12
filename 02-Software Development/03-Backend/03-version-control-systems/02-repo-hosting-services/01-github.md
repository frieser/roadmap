---
---

# GitHub

GitHub is the industry-standard code hosting platform that has evolved into a complete **DevOps ecosystem**. For backend developers, it is not just about storing code but managing the entire software lifecycle through Actions, Packages, and Security tools.

## Key Features

### 1. GitHub Actions (CI/CD)
The native automation engine.
- **Workflows**: Defined in `.github/workflows/*.yml`.
- **Runners**:
    - **GitHub-hosted**: Standard VMs (Ubuntu, Windows, macOS).
    - **Self-hosted**: Your own infrastructure (essential for VPC access or heavy builds).
- **Composite vs Reusable**:
    - **Composite Actions**: Combine steps for local reuse.
    - **Reusable Workflows**: Call entire pipeline definitions across repos.

### 2. Security (Shift Left)
- **Dependabot**: Automated dependency updates and vulnerability patching.
- **Secret Scanning**: Blocks commits containing known credential formats (AWS keys, tokens).
- **CodeQL**: Semantic analysis engine to find vulnerabilities (SQLi, XSS) in code.

### 3. CLI (`gh`)
Control GitHub from the terminal.
```bash
# Create a PR from current branch
gh pr create --title "feat: auth" --body "Adds JWT support" --base main

# Check status of actions
gh run list
```

## Backend Usage Patterns

### Go Module CI Pipeline
Example for a Go backend service:
```yaml
name: Go Build & Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.24'
      # Private Modules Support
      - name: Configure Private Modules
        run: git config --global url."https://${{ secrets.GH_TOKEN }}@github.com/".insteadOf "https://github.com/"
      - name: Test
        run: go test -v -race ./...
```

### Package Registry
Publish Docker images or Go modules directly to GitHub Packages (`ghcr.io`).

## Interview Questions
1. **Composite Action vs Reusable Workflow?**
   - *Composite*: Steps sharing the same job/runner. *Reusable*: Independent workflows with their own jobs/runners.
2. **How to handle secrets?**
   - Use Repository Secrets for static keys and OIDC for cloud provider authentication (AWS/GCP/Azure) to avoid long-lived credentials.
3. **What is "Push Protection"?**
   - Pre-receive hook in Secret Scanning that rejects commits containing detected secrets.
