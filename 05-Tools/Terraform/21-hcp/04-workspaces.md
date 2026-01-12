---
tags: ['tools', 'roadmap']
---

## Summary
In Terraform Cloud and Enterprise, a **Workspace** is the primary unit of organization. Unlike the Open Source CLI where a workspace is just a separate state file, a TFC/TFE workspace includes the infrastructure configuration (VCS or API), persistent state, variable definitions, and a history of all runs. Workspaces are managed within **Projects** to provide a hierarchical structure for large organizations.

## Detailed Explanation

### 1. Workspace vs. CLI Directory
In local development, one directory usually corresponds to one state. In TFC:
- A Workspace tracks **State Versioning**: Every apply saves a history of state changes.
- **Locking**: TFC handles state locking automatically to prevent concurrent runs.
- **Remote Execution**: Terraform runs (plan/apply) happen on HashiCorp-managed infrastructure or private agents.

### 2. Execution Modes
Workspaces can run in three different modes:
- **Remote**: Everything runs on HashiCorp infrastructure.
- **Local**: TFC only stores the state; plan/apply runs on your local CLI.
- **Agent**: Runs happen on your own infrastructure (private network) using a lightweight **TFC Agent**. This allows TFC to manage resources behind a firewall.

### 3. Hierarchical Organization (Projects)
Workspaces are grouped into **Projects**.
- **Access Control**: Permissions can be granted at the project level, automatically applying to all workspaces within it.
- **Variable Sets**: Can be scoped to a specific project.

### 4. State Sharing
Workspaces often need outputs from other workspaces (e.g., an App workspace needing the VPC ID from a Network workspace).
- **Remote State Sharing**: You must explicitly authorize a workspace to read another's state outputs for security.

### HCL Configuration Example

**Managing Workspaces and Projects with the `tfe` Provider:**
```hcl
# 1. Create a Project for the Engineering Team
resource "tfe_project" "eng_team" {
  organization = "my-org"
  name         = "Engineering-Platform"
}

# 2. Create a Workspace within that Project
resource "tfe_workspace" "prod_db" {
  name              = "production-database"
  organization      = "my-org"
  project_id        = tfe_project.eng_team.id
  terraform_version = "1.6.0"
  
  execution_mode    = "remote"
  auto_apply        = false # Requires manual approval
  
  vcs_repo {
    identifier     = "my-org/infra-repo"
    oauth_token_id = "ot-xxxxxx"
  }
}

# 3. Allow another workspace to consume this state
resource "tfe_workspace" "app_server" {
  name                      = "app-server"
  organization              = "my-org"
  remote_state_consumer_ids = [tfe_workspace.prod_db.id]
}
```

### Go Application
The `go-tfe` SDK allows for programmatic workspace management, useful for building "vending machines" that provision infrastructure environments for developers.

```go
package main

import (
	"context"
	"fmt"
	"github.com/hashicorp/go-tfe"
)

func main() {
	client, _ := tfe.NewClient(&tfe.Config{Token: "TFC_TOKEN"})
	
	// Create a workspace programmatically
	ws, err := client.Workspaces.Create(context.Background(), "my-org", tfe.WorkspaceCreateOptions{
		Name: tfe.String("new-dev-env"),
	})
	if err != nil {
		panic(err)
	}
	fmt.Printf("Workspace created: %s (ID: %s)\n", ws.Name, ws.ID)
}
```

## Interview Questions

**Q: How does a TFC Workspace differ from a standard CLI workspace?**
**A:** A CLI workspace is essentially just a separate state file in the same directory. A TFC Workspace is a robust environment that includes the state, but also links to a VCS repository, manages its own variables, maintains a full run history, and provides a UI for collaboration.

**Q: When would you use "Agent" execution mode?**
**A:** Agent mode is used when Terraform needs to provision resources inside a private network or on-premises data center that the public TFC infrastructure cannot reach. You deploy a TFC Agent inside your network, and it polls TFC for work.

**Q: What are "Projects" in Terraform Cloud?**
**A:** Projects are a way to group related workspaces. They help organize large numbers of workspaces and allow for simplified permission management and variable set scoping.

**Q: How do you share data between two workspaces?**
**A:** Using the `terraform_remote_state` data source. However, in TFC, you must also configure the "Remote State Sharing" settings on the source workspace to authorize the consumer workspace to read its state.
