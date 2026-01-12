---
tags: ['tools', 'roadmap']
---

## Summary
VCS (Version Control System) Integration is the cornerstone of the **VCS-driven workflow** in Terraform Cloud and Enterprise. It allows Terraform to automatically trigger infrastructure runs based on code changes. It supports major providers like GitHub, GitLab, and Bitbucket, offering features like **Speculative Plans** on Pull Requests and advanced **Monorepo** management via directory filtering.

## Detailed Explanation

### 1. Connection Methods
- **GitHub App**: The modern, recommended way for GitHub.com. It offers superior security and more granular permissions.
- **OAuth**: The traditional method for GitLab, Bitbucket, and Azure DevOps. Requires setting up an OAuth application in the VCS provider.

### 2. VCS-Driven Runs
- **Push Triggers**: A commit to the tracked branch (e.g., `main`) triggers a "Plan & Apply" run.
- **Speculative Plans**: When a Pull Request is opened, TFC runs a `terraform plan` and posts the results back to the PR as a comment or status check. This allows for peer review of infrastructure changes before merging.

### 3. Monorepo Support
In a monorepo, many environments or services live in one repository. TFC manages this via:
- **Working Directory**: Tells TFC which folder to execute Terraform in (e.g., `envs/prod`).
- **Trigger Patterns**: Glob patterns that define which file changes should trigger a run. If a change occurs in an unrelated folder, the run is skipped, saving resources and preventing unnecessary noise.

### 4. Automatic Run Canceling
If multiple commits are pushed in quick succession, TFC can automatically cancel older, pending runs in favor of the latest commit, ensuring you aren't deploying outdated configurations.

### HCL Configuration Example

**Configuring a Workspace with VCS and Monorepo Settings:**
```hcl
data "tfe_oauth_client" "github" {
  organization = "my-org"
  name         = "github-oauth-client"
}

resource "tfe_workspace" "api_prod" {
  name         = "api-service-production"
  organization = "my-org"

  # MONOREPO SETTINGS
  working_directory = "terraform/services/api"
  
  # Trigger only if code in the service or shared modules changes
  trigger_patterns = [
    "terraform/services/api/**/*",
    "terraform/modules/shared/**/*"
  ]

  # VCS CONFIGuration
  vcs_repo {
    identifier     = "my-org/infrastructure-monorepo"
    branch         = "main"
    oauth_token_id = data.tfe_oauth_client.github.oauth_token_id
  }

  speculative_enabled = true # Enable PR plans
}
```

### Go Application
Automation tools often need to fetch the current VCS configuration of a workspace to audit security settings.

```go
package main

import (
	"context"
	"fmt"
	"github.com/hashicorp/go-tfe"
)

func main() {
	client, _ := tfe.NewClient(&tfe.Config{Token: "TFC_TOKEN"})
	ws, _ := client.Workspaces.Read(context.Background(), "my-org", "my-workspace")

	if ws.VCSRepo != nil {
		fmt.Printf("Workspace linked to: %s (Branch: %s)\n", 
            ws.VCSRepo.Identifier, ws.VCSRepo.Branch)
	} else {
		fmt.Println("Workspace is not linked to VCS (CLI/API driven)")
	}
}
```

## Interview Questions

**Q: What is a "Speculative Plan" in Terraform Cloud?**
**A:** It is a plan triggered by a Pull Request or Merge Request. It shows what changes *would* happen if the PR was merged, but it never allows an `apply`. The results are shared as a status check or comment on the PR to facilitate code review.

**Q: How does TFC handle Monorepos to avoid triggering every workspace on every commit?**
**A:** By using the `working_directory` and `trigger_patterns` settings. `working_directory` sets the scope for execution, and `trigger_patterns` (globs) ensure a run only starts if relevant files within those patterns have actually changed.

**Q: What are the advantages of the GitHub App integration over OAuth?**
**A:** The GitHub App is easier to set up, doesn't require a dedicated service account, provides more granular permissions (repository-specific), and supports faster webhook processing.

**Q: Can you trigger a run in TFC without using VCS?**
**A:** Yes. You can use the **CLI-driven workflow** (running `terraform apply` locally while using TFC as a backend) or the **API-driven workflow** (uploading a configuration archive directly to the TFC API).
