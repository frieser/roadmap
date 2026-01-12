# Bitbucket

## Summary
Bitbucket is a Git-based source code repository hosting service owned by Atlassian. It is uniquely positioned for teams already using the Atlassian suite (Jira, Confluence, Trello), offering deep integration with Jira for issue tracking. It provides Bitbucket Pipelines for CI/CD and is available in Cloud and Data Center versions.

## Detailed Explanation
Bitbucket is often chosen by large enterprises because of its integration with Atlassian's project management tools.

### Key Features
*   **Jira Integration:** You can see build status, deployments, and PRs directly inside Jira issues. Branching can also be initiated from a Jira ticket.
*   **Bitbucket Pipelines:** A CI/CD service built into Bitbucket Cloud. Configuration is defined in `bitbucket-pipelines.yml`.
*   **Branch Permissions:** Granular control over who can write to or merge into specific branches, often more detailed than GitHub's free tier.
*   **Git LFS Support:** Strong support for Git Large File Storage, useful for projects with large binary assets.

### Workspace/Project Hierarchy
Bitbucket uses a "Workspace" -> "Project" -> "Repository" structure, which helps large organizations group related repositories under single projects for better access control.

## Go-specific Context
Bitbucket Pipelines supports Go out of the box using Docker-based build environments.

*   **Integrated Testing:** Running Go tests within the pipeline and displaying results in the Bitbucket UI.
*   **Artifact Management:** Using Bitbucket's artifact system to pass compiled Go binaries between pipeline steps.
*   **Authentication:** Using Bitbucket "App Passwords" or SSH keys to allow `go get` to fetch private dependencies from other Bitbucket repositories.

```yaml
# Example bitbucket-pipelines.yml for Go
image: golang:1.23

pipelines:
  default:
    - step:
        name: Build and Test
        script:
          - go build ./...
          - go test -v ./...
        artifacts:
          - bin/**
```

## Interview Questions
**Q: What is the main advantage of Bitbucket for teams using Jira?**
**A:** The main advantage is the deep, native integration. Development activity (commits, branches, PRs) is automatically linked to Jira tickets, providing stakeholders with real-time visibility into the development progress without leaving Jira.

**Q: What are Bitbucket Pipelines?**
**A:** Bitbucket Pipelines is an integrated CI/CD tool that allows you to automate the build, test, and deployment of your code. It runs your code in Docker containers on Atlassian's infrastructure, configured by a `bitbucket-pipelines.yml` file.

**Q: Explain Bitbucket "App Passwords".**
**A:** App Passwords are unique tokens used for authenticating with the Bitbucket API or Git over HTTPS when two-step verification is enabled. They are safer than using your main account password because they can be scoped to specific permissions and revoked individually.
