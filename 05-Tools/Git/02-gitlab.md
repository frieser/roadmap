# GitLab

## Summary
GitLab is a comprehensive DevSecOps platform that provides Git repository hosting, CI/CD, issue tracking, and security scanning in a single application. Unlike GitHub, GitLab offers a robust self-hosted version, making it a popular choice for enterprises with strict data sovereignty and security requirements.

## Detailed Explanation
GitLab's philosophy is "everything in one place." It covers the entire software development lifecycle (SDLC).

### Key Features
*   **Integrated CI/CD:** GitLab CI/CD is considered one of the most powerful and flexible built-in CI tools. Configuration is done via `.gitlab-ci.yml`.
*   **Merge Requests (MRs):** Equivalent to GitHub's Pull Requests, but often includes integrated security and performance reports.
*   **GitLab Runners:** Lightweight agents that run the CI/CD jobs. They can be hosted on-premises, in the cloud, or on developer machines.
*   **Auto DevOps:** A feature that automatically configures CI/CD based on best practices for your language and framework.

### Self-Hosting
GitLab's Community Edition (CE) allows organizations to host their own GitLab instance, providing full control over the infrastructure and data.

## Go-specific Context
GitLab provides excellent support for Go, especially for containerized applications.

*   **Container Registry:** GitLab includes a built-in Docker registry, making it easy to build and store Docker images for Go microservices.
*   **Dependency Proxy:** Helps speed up builds by caching Go modules and Docker images.
*   **Go Vulnerability Scanning:** Integrated security tools can scan Go binaries and dependencies for known vulnerabilities as part of the pipeline.

```yaml
# Example .gitlab-ci.yml for a Go project
image: golang:1.23

stages:
  - build
  - test

compile:
  stage: build
  script:
    - go build -o app main.go
  artifacts:
    paths:
      - app

run_tests:
  stage: test
  script:
    - go test ./...
```

## Interview Questions
**Q: What is a GitLab Runner?**
**A:** A GitLab Runner is an application that works with GitLab CI/CD to run jobs in a pipeline. It can be installed on various operating systems and uses "executors" (like Docker, Shell, or VirtualBox) to run the scripts defined in `.gitlab-ci.yml`.

**Q: How does GitLab's CI/CD configuration differ from GitHub's?**
**A:** GitLab uses a single `.gitlab-ci.yml` file at the root of the repo to define the entire pipeline, stages, and jobs. GitHub Actions uses multiple YAML files in `.github/workflows/`, where each file represents a separate workflow. GitLab's system is often seen as more centralized for complex pipelines.

**Q: What is the GitLab Container Registry?**
**A:** It is a secure, private registry for Docker images built into GitLab. It allows developers to build, push, and pull images directly from their GitLab CI/CD pipelines, simplifying the deployment of containerized Go applications.
