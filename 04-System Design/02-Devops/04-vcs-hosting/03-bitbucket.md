---
---

# Bitbucket for DevOps

Bitbucket is Atlassian's Git solution, designed for professional teams. Its tight integration with **Jira** (project management) and **Confluence** (documentation) makes it a top choice for enterprises already in the Atlassian ecosystem.

## Summary

Bitbucket offers **Bitbucket Pipelines** for CI/CD, which allows configuration via a `bitbucket-pipelines.yml` file. Its key differentiator is the seamless workflow between code and issue tracking: you can create branches directly from Jira tickets, and Jira tickets automatically update status based on Bitbucket commits and deployments.

## Detailed Explanation

### 1. Bitbucket Pipelines
*   **Container-Native**: Every step in a pipeline runs inside a Docker container. You specify the image at the top level or per step.
*   **bitbucket-pipelines.yml**: The configuration file.
*   **Pipes**: Reusable chunks of configuration (similar to GitHub Actions) that make it easy to integrate with third-party tools (AWS, Slack, SonarCloud) by just pasting a snippet.

### 2. Integration with Jira
*   **Smart Commits**: Developers can transition Jira issues by including commands in commit messages (e.g., `JIRA-123 #resolve #comment Fixed the login bug`).
*   **Deployment Tracking**: The "Deployments" dashboard in Bitbucket syncs with Jira to show exactly which issues are deployed to Staging or Production.

### 3. Branching Model
Bitbucket is often associated with the classic **Gitflow** workflow, although it fully supports trunk-based development. It provides GUI controls for managing branch permissions and merge strategies (squash, fast-forward).

---

## Go Implementation Example

To interact with Bitbucket Cloud, the `ktrysmt/go-bitbucket` library is commonly used. It handles authentication (Basic Auth or OAuth) and provides struct-based access to repositories and pull requests.

```go
package main

import (
	"fmt"
	"log"

	"github.com/ktrysmt/go-bitbucket"
)

func main() {
	// 1. Authenticate using App Password
	// Bitbucket Cloud requires username + app password for API access
	client := bitbucket.NewBasicAuth("my-username", "my-app-password")

	// 2. Get Repository Details
	repoOpts := &bitbucket.RepositoryOptions{
		Owner:    "my-workspace",
		RepoSlug: "my-backend-service",
	}
	
	repo, err := client.Repositories.Repository.Get(repoOpts)
	if err != nil {
		log.Fatal("Error fetching repo:", err)
	}
	fmt.Printf("Repository: %s (Language: %s)\n", repo.Name, repo.Language)

	// 3. List Pull Requests
	prOpts := &bitbucket.PullRequestsOptions{
		Owner:    "my-workspace",
		RepoSlug: "my-backend-service",
		State:    "OPEN",
	}
	
	prs, err := client.Repositories.PullRequests.List(prOpts)
	if err != nil {
		log.Fatal(err)
	}

	// The library returns interface{}, so we often need to inspect the structure
	// or cast it, though the response generally contains a list of PR objects.
	fmt.Printf("Found %d open PRs\n", len(prs.(map[string]interface{})["values"].([]interface{})))

	// 4. Create a Pull Request (simplified)
	/*
	createOpts := &bitbucket.PullRequestsOptions{
		Owner: "my-workspace",
		RepoSlug: "my-backend-service",
		Title: "Refactor API",
		Source: bitbucket.PullRequestBranch{Branch: "feature/api"},
		Destination: bitbucket.PullRequestBranch{Branch: "main"},
	}
	_, err = client.Repositories.PullRequests.Create(createOpts)
	*/
}
```

## Interview Questions

**Q: What are "Pipes" in Bitbucket Pipelines?**
**A:** Pipes are pre-configured Docker containers that simplify complex tasks in `bitbucket-pipelines.yml`. Instead of writing a 20-line script to deploy to AWS S3, you can use the `atlassian/aws-s3-deploy` pipe and just pass it your keys and bucket name. They are analogous to "Actions" in GitHub.

**Q: How does Bitbucket's integration with Jira improve the DevOps feedback loop?**
**A:** It links code to business value. By referencing Jira tickets in commits, the entire team (including non-technical product owners) can see the status of a feature—from "In Progress" to "Merged" to "Deployed"—directly within the Jira board, without needing to ask developers or check the git history.

**Q: Can you use custom Docker images in Bitbucket Pipelines?**
**A:** Yes, Bitbucket Pipelines is Docker-native. You can specify `image: golang:1.21` at the top of your YAML file, and all script commands will execute inside that container. You can also specify different images for different steps if needed (e.g., one step uses `node:18` to build frontend, the next uses `golang:1.21` for backend).
